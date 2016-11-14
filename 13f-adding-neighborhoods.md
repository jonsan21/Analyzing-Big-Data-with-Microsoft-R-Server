---
layout: page
title: Overview of RevoScaleR
---
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

In order to run the above transformation on the whole data, we need to take the above code and wrap it into a transformation function that we can pass to `rxDataStep` through the `transfromFunc` argument. We call the transformation function `find_nhoods`. Our transformation function will add both pick-up neighborhood and borough as well as drop-off neighborhood and borough.

```R
find_nhoods <- function(data) {
  
  # extract pick-up lat and long and find their neighborhoods
  pickup_longitude <- ifelse(is.na(data$pickup_longitude), 0, data$pickup_longitude)
  pickup_latitude <- ifelse(is.na(data$pickup_latitude), 0, data$pickup_latitude)
  data_coords <- data.frame(long = pickup_longitude, lat = pickup_latitude)
  coordinates(data_coords) <- c('long', 'lat')
  nhoods <- over(data_coords, shapefile)
  
  ## add only the pick-up neighborhood and city columns to the data
  data$pickup_nhood <- nhoods$NAME
  data$pickup_borough <- nhoods$CITY
  
  # extract drop-off lat and long and find their neighborhoods
  dropoff_longitude <- ifelse(is.na(data$dropoff_longitude), 0, data$dropoff_longitude)
  dropoff_latitude <- ifelse(is.na(data$dropoff_latitude), 0, data$dropoff_latitude)
  data_coords <- data.frame(long = dropoff_longitude, lat = dropoff_latitude)
  coordinates(data_coords) <- c('long', 'lat')
  nhoods <- over(data_coords, shapefile)
  
  ## add only the drop-off neighborhood and city columns to the data  
  data$dropoff_nhood <- nhoods$NAME
  data$dropoff_borough <- nhoods$CITY
  
  ## return the data with the new columns added in
  data
}
```

Now that we've got our function, it's time to test it. We can do so by running `rxDataStep` on the sample data `nyc_sample_df`. This is a good way to make sure the transformation works before we run it on the large XDF file `nyc_xdf`. Sometimes errors messages we get are more informative when we apply the transformation to a `data.frame`, and it's easier to trace it back and debug it.  So here we use `rxDataStep` and feed it `nyc_sample_df` (with no `outFile` argument) to see what the data looks like after applying the transformation function `find_nhoods` to it.

```R
# test the function on a data.frame using rxDataStep
head(rxDataStep(nyc_sample_df, transformFunc = find_nhoods, transformPackages = c("sp", "maptools"), 
                transformObjects = list(shapefile = mht_shapefile)))
```

```Rout
  VendorID tpep_pickup_datetime tpep_dropoff_datetime passenger_count trip_distance
1        2  2016-01-01 00:00:00   2016-01-01 00:00:00               2          1.10
2        2  2016-01-01 00:00:00   2016-01-01 00:00:00               5          4.90
3        2  2016-01-01 00:00:00   2016-01-01 00:00:00               1         10.54
4        2  2016-01-01 00:00:00   2016-01-01 00:00:00               1          4.75
5        2  2016-01-01 00:00:00   2016-01-01 00:00:00               3          1.76
6        2  2016-01-01 00:00:00   2016-01-01 00:18:30               2          5.52
  pickup_longitude pickup_latitude RatecodeID store_and_fwd_flag dropoff_longitude
1        -73.99037        40.73470          1                  N         -73.98184
2        -73.98078        40.72991          1                  N         -73.94447
3        -73.98455        40.67957          1                  N         -73.95027
4        -73.99347        40.71899          1                  N         -73.96224
5        -73.96062        40.78133          1                  N         -73.97726
6        -73.98012        40.74305          1                  N         -73.91349
  dropoff_latitude payment_type fare_amount extra mta_tax tip_amount tolls_amount
1         40.73241            2         7.5   0.5     0.5          0            0
2         40.71668            1        18.0   0.5     0.5          0            0
3         40.78893            1        33.0   0.5     0.5          0            0
4         40.65733            2        16.5   0.0     0.5          0            0
5         40.75851            2         8.0   0.0     0.5          0            0
6         40.76314            2        19.0   0.5     0.5          0            0
  improvement_surcharge total_amount      pickup_nhood          pickup_borough
1                   0.3          8.8 Greenwich Village New York City-Manhattan
2                   0.3         19.3      East Village New York City-Manhattan
3                   0.3         34.3       Boerum Hill  New York City-Brooklyn
4                   0.3         17.3   Lower East Side New York City-Manhattan
5                   0.3          8.8   Upper East Side New York City-Manhattan
6                   0.3         20.3          Gramercy New York City-Manhattan
             dropoff_nhood         dropoff_borough
1                 Gramercy New York City-Manhattan
2                     <NA>                    <NA>
3                Yorkville New York City-Manhattan
4                     <NA>                    <NA>
5                  Midtown New York City-Manhattan
6 Astoria-Long Island City    New York City-Queens
```

The last four columns in the data correspond to the neighborhood columns we wanted. So we are fairly confident now that the transformation will work when we run it on `nyc_xdf`.

```R
st <- Sys.time()
rxDataStep(nyc_xdf, nyc_xdf, overwrite = TRUE, transformFunc = find_nhoods, transformPackages = c("sp", "maptools", "rgeos"), 
           transformObjects = list(shapefile = mht_shapefile))
Sys.time() - st
rxGetInfo(nyc_xdf, numRows = 5)
```

```Rout
Time difference of 30.77251 mins

File name: C:\Data\NYC_taxi\yellow_tripdata_2016.xdf 
Number of observations: 69406520 
Number of variables: 29 
Number of blocks: 141 
Compression type: zlib 
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
  improvement_surcharge total_amount tip_percent pickup_hour pickup_dow
1                   0.3          8.8           0    10PM-1AM        Fri
2                   0.3         19.3           0    10PM-1AM        Fri
3                   0.3         34.3           0    10PM-1AM        Fri
4                   0.3         17.3           0    10PM-1AM        Fri
5                   0.3          8.8           0    10PM-1AM        Fri
  dropoff_hour dropoff_dow trip_duration      pickup_nhood          pickup_borough
1     10PM-1AM         Fri             0 Greenwich Village New York City-Manhattan
2     10PM-1AM         Fri             0      East Village New York City-Manhattan
3     10PM-1AM         Fri             0       Boerum Hill  New York City-Brooklyn
4     10PM-1AM         Fri             0   Lower East Side New York City-Manhattan
5     10PM-1AM         Fri             0   Upper East Side New York City-Manhattan
  dropoff_nhood         dropoff_borough
1      Gramercy New York City-Manhattan
2          <NA>                    <NA>
3     Yorkville New York City-Manhattan
4          <NA>                    <NA>
5       Midtown New York City-Manhattan
```