---
layout: page
title: Overview of RevoScaleR
---
# Checking column types

It's a good idea to check our column types and make sure nothing strange stands out. In addition to column types, the `rxGetInfo` function also shows lows and highs for numeric columns, which can be useful for identifying outliers or running sanity checks. With the `numRows = 10` argument, we can look at the first 10 rows of the data as well.

```R
rxGetInfo(nyc_xdf, getVarInfo = TRUE, numRows = 5) # show column types and the first 10 rows
```

```Rout
File name: C:\Data\NYC_taxi\yellow_tripdata_2016.xdf 
Number of observations: 69406520 
Number of variables: 19 
Number of blocks: 141 
Compression type: zlib 
Variable information: 

Var 1: VendorID 2 factor levels: 2 1
Var 2: tpep_pickup_datetime, Type: character
Var 3: tpep_dropoff_datetime, Type: character
Var 4: passenger_count, Type: integer, Low/High: (0, 9)
Var 5: trip_distance, Type: numeric, Low/High: (-3390583.8000, 19072628.8000)
Var 6: pickup_longitude, Type: numeric, Low/High: (-165.0819, 118.4089)
Var 7: pickup_latitude, Type: numeric, Low/High: (-77.0395, 66.8568)
Var 8: RatecodeID, Type: integer, Low/High: (1, 99)
Var 9: store_and_fwd_flag 2 factor levels: N Y
Var 10: dropoff_longitude, Type: numeric, Low/High: (-161.6987, 106.2469)
Var 11: dropoff_latitude, Type: numeric, Low/High: (-77.0395, 405.3167)
Var 12: payment_type 5 factor levels: 2 1 3 4 5
Var 13: fare_amount, Type: numeric, Low/High: (-957.6000, 628544.7400)
Var 14: extra, Type: numeric, Low/High: (-58.5000, 648.8700)
Var 15: mta_tax, Type: numeric, Low/High: (-2.7000, 89.7000)
Var 16: tip_amount, Type: numeric, Low/High: (-220.8000, 998.1400)
Var 17: tolls_amount, Type: numeric, Low/High: (-99.9900, 1410.3200)
Var 18: improvement_surcharge, Type: numeric, Low/High: (-0.3000, 11.6400)
Var 19: total_amount, Type: numeric, Low/High: (-958.4000, 629033.7800)

Data (5 rows starting with row 1):
  VendorID tpep_pickup_datetime tpep_dropoff_datetime passenger_count trip_distance
1        2  2016-01-01 00:00:00   2016-01-01 00:00:00               2          1.10
2        2  2016-01-01 00:00:00   2016-01-01 00:00:00               5          4.90
3        2  2016-01-01 00:00:00   2016-01-01 00:00:00               1         10.54
4        2  2016-01-01 00:00:00   2016-01-01 00:00:00               1          4.75
5        2  2016-01-01 00:00:00   2016-01-01 00:00:00               3          1.76
  pickup_longitude pickup_latitude RatecodeID store_and_fwd_flag dropoff_longitude
1        -73.99037        40.73470          1                  N         -73.98184
2        -73.98078        40.72991          1                  N         -73.94447
3        -73.98455        40.67957          1                  N         -73.95027
4        -73.99347        40.71899          1                  N         -73.96224
5        -73.96062        40.78133          1                  N         -73.97726
  dropoff_latitude payment_type fare_amount extra mta_tax tip_amount tolls_amount
1         40.73241            2         7.5   0.5     0.5          0            0
2         40.71668            1        18.0   0.5     0.5          0            0
3         40.78893            1        33.0   0.5     0.5          0            0
4         40.65733            2        16.5   0.0     0.5          0            0
5         40.75851            2         8.0   0.0     0.5          0            0
  improvement_surcharge total_amount
1                   0.3          8.8
2                   0.3         19.3
3                   0.3         34.3
4                   0.3         17.3
5                   0.3          8.8
```
