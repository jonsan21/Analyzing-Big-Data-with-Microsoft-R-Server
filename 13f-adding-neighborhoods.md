# Finding neighborhoods

In the last section, we stored the shapefile for Manhattan in an object called `mht_shapefile`, which we than used to plot a map of Manhattan neighborhoods. Let's now see how we can use the same shapefile along with the function `over` (part of the `sp` package) to find out pick-up and drop-off neighborhoods for each trip. We will demonstrate how we can do that with a `data.frame` (using the sample of the NYC taxi data we took earlier) and leave it as an exercise to find out how we can wrap the code in a transformation function and apply it to the XDF data.

To add pick-up neighborhood columns to `nyc_sample_df`, we need to do as follows:
  - replace NAs in `pickup_latitude` and `pickup_longitude` with 0, otherwise we will get an error
  - use the `coordinates` function to specify that the above two columns represent the geographical coordinates in the data
  - use the `over` function to get neighborhoods based on the coordinates specified above, which returns a `data.frame` with neighborhood information
  - attach the results to the original data using `cbind` after giving them relevant column names

```R
# take only the coordinate columns, and replace NAs with 0
data_coords <- transmute(nyc_sample_df,
  long = ifelse(is.na(pickup_longitude), 0, pickup_longitude),
  lat = ifelse(is.na(pickup_latitude), 0, pickup_latitude)
)
# we specify the columns that correspond to the coordinates
coordinates(data_coords) <- c('long', 'lat')
# returns the neighborhoods based on coordinates
nhoods <- over(data_coords, mht_shapefile)
# rename the column names in nhoods
names(nhoods) <- paste('pickup', tolower(names(nhoods)), sep = '_')
# combine the neighborhood information with the original data
nyc_sample_df <- cbind(nyc_sample_df, nhoods[, grep('name|city', names(nhoods))])
head(nyc_sample_df)
```

```Rout
  VendorID tpep_pickup_datetime tpep_dropoff_datetime passenger_count
1        2  2016-01-01 00:00:00   2016-01-01 00:00:00               2
2        2  2016-01-01 00:00:00   2016-01-01 00:00:00               5
3        2  2016-01-01 00:00:00   2016-01-01 00:00:00               1
4        2  2016-01-01 00:00:00   2016-01-01 00:00:00               1
5        2  2016-01-01 00:00:00   2016-01-01 00:00:00               3
6        2  2016-01-01 00:00:00   2016-01-01 00:18:30               2
  trip_distance pickup_longitude pickup_latitude RatecodeID store_and_fwd_flag
1          1.10        -73.99037        40.73470          1                  N
2          4.90        -73.98078        40.72991          1                  N
3         10.54        -73.98455        40.67957          1                  N
4          4.75        -73.99347        40.71899          1                  N
5          1.76        -73.96062        40.78133          1                  N
6          5.52        -73.98012        40.74305          1                  N
  dropoff_longitude dropoff_latitude payment_type fare_amount extra mta_tax
1         -73.98184         40.73241            2         7.5   0.5     0.5
2         -73.94447         40.71668            1        18.0   0.5     0.5
3         -73.95027         40.78893            1        33.0   0.5     0.5
4         -73.96224         40.65733            2        16.5   0.0     0.5
5         -73.97726         40.75851            2         8.0   0.0     0.5
6         -73.91349         40.76314            2        19.0   0.5     0.5
  tip_amount tolls_amount improvement_surcharge total_amount
1          0            0                   0.3          8.8
2          0            0                   0.3         19.3
3          0            0                   0.3         34.3
4          0            0                   0.3         17.3
5          0            0                   0.3          8.8
6          0            0                   0.3         20.3
              pickup_city       pickup_name
1 New York City-Manhattan Greenwich Village
2 New York City-Manhattan      East Village
3                    <NA>              <NA>
4 New York City-Manhattan   Lower East Side
5 New York City-Manhattan   Upper East Side
6 New York City-Manhattan          Gramercy
```