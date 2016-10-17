# Examining neighborhoods

By passing `~ .` as the formula to `rxSummary`, we can summarize all the columns in the data.

```R
system.time(
rxs_all <- rxSummary( ~ ., nyc_xdf)
)
```

```Rout
Rows Processed: 69406520 
   user  system elapsed 
   0.05    0.02   85.16 
```

For example, the numeric summaries for the relevant columns in the data are stored in `rxs_all` under the element called `sDataFrame`.

```R
head(rxs_all$sDataFrame)
```

```Rout
                   Name       Mean      StdDev           Min           Max ValidObs
1              VendorID         NA          NA            NA            NA 69406520
2  tpep_pickup_datetime         NA          NA            NA            NA        0
3 tpep_dropoff_datetime         NA          NA            NA            NA        0
4       passenger_count   1.660674    1.310478        0.0000        9.0000 69406520
5         trip_distance   4.850022 4044.503422 -3390583.8000 19072628.8000 69406520
6      pickup_longitude -72.920469    8.763351     -165.0819      118.4089 69406520
  MissingObs
1          0
2          0
3          0
4          0
5          0
6          0
```

If we wanted one-way tables showing counts of levels for each `factor` column in the data, we can refer to `rxs_all` to obtain that, but if we need to get two-way tables showing counts of combinations of certain `factor` columns with others we need to pass the correct formula to the summary function. Here we use `rxCrossTabs` to get the number of trips from one neighborhood going into another.

```R
nhoods_by_borough <- rxCrossTabs( ~ pickup_nhood:pickup_borough, nyc_xdf)
nhoods_by_borough <- nhoods_by_borough$counts[[1]]
nhoods_by_borough <- as.data.frame(nhoods_by_borough)

# get the neighborhoods by borough
lnbs <- lapply(names(nhoods_by_borough), function(vv) subset(nhoods_by_borough, nhoods_by_borough[ , vv] > 0, select = vv, drop = FALSE))
lapply(lnbs, head)
```

```Rout
[[1]]
[1] Albany
<0 rows> (or 0-length row.names)

[[2]]
[1] Buffalo
<0 rows> (or 0-length row.names)

[[3]]
             New York City-Bronx
Baychester                   125
Bedford Park                1413
City Island                   52
Country Club                 354
Eastchester                   98
Fordham                     1243

[[4]]
                   New York City-Brooklyn
Bay Ridge                            3378
Bedford-Stuyvesant                  54269
Bensonhurst                          1159
Boerum Hill                         76404
Borough Park                         8762
Brownsville                          2757

[[5]]
              New York City-Manhattan
Battery Park                   643283
Carnegie Hill                  807204
Central Park                   936840
Chelsea                       4599098
Chinatown                      211229
Clinton                       2050545

[[6]]
                         New York City-Queens
Astoria-Long Island City               303231
Auburndale                                464
Clearview                                 152
College Point                               1
Corona                                   1496
Douglastown-Little Neck                   937

[[7]]
                            New York City-Staten Island
Annandale                                             6
Ardon Heights                                        22
Bloomfield-Chelsea-Travis                            26
Charlestown-Richmond Valley                           7
Clifton                                             525
Ettingville                                          13

[[8]]
[1] Rochester
<0 rows> (or 0-length row.names)

[[9]]
[1] Syracuse
<0 rows> (or 0-length row.names)
```