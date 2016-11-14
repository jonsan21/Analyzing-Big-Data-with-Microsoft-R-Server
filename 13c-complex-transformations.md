---
layout: page
title: Overview of RevoScaleR
---
# Complex transformations

In the last part `rxDataStep` was introduced to perform a simple one-liner transformation.  We now use `rxDataStep` again to perform some other, this time more complicated transformations.  We can sometimes perform these more complex transformations as longer one-liners using the `transforms` argument, following the above example.  But a cleaner way to do it is to create a function that contains the logic of our transformations and pass it to the `transformFunc` argument. This function takes the data as input and usually returns the same data as output with one or more columns added or modified. More specifically, the input to the transformation function is a `list` whose elements are the columns.  Otherwise, it is just like any R function. Using the `transformFunc` argument allows us to focus on writing a transformation function and quickly testing them on the sample `data.frame` before we run them on the whole data.

For the NYC Taxi data, we are interested in comparing trips based on day of week and the time of day.  Those two columns do not exist yet, but we can extract them from pick-up date and time and drop-off date and time.  To extract the above features, we use the `lubridate` package, which has useful functions for dealing with date and time columns.  To perform these transformations, we use a transformation function called `xforms`.

```R
xforms <- function(data) { # transformation function for extracting some date and time features
  
  weekday_labels <- c('Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat')
  cut_levels <- c(1, 5, 9, 12, 16, 18, 22)
  hour_labels <- c('1AM-5AM', '5AM-9AM', '9AM-12PM', '12PM-4PM', '4PM-6PM', '6PM-10PM', '10PM-1AM')
  
  pickup_datetime <- ymd_hms(data$tpep_pickup_datetime, tz = "UTC")
  pickup_hour <- addNA(cut(hour(pickup_datetime), cut_levels))
  pickup_dow <- factor(wday(pickup_datetime), levels = 1:7, labels = weekday_labels)
  levels(pickup_hour) <- hour_labels
  
  dropoff_datetime <- ymd_hms(data$tpep_dropoff_datetime, tz = "UTC")
  dropoff_hour <- addNA(cut(hour(dropoff_datetime), cut_levels))
  dropoff_dow <- factor(wday(dropoff_datetime), levels = 1:7, labels = weekday_labels)
  levels(dropoff_hour) <- hour_labels
  
  data$pickup_hour <- pickup_hour
  data$pickup_dow <- pickup_dow
  data$dropoff_hour <- dropoff_hour
  data$dropoff_dow <- dropoff_dow
  data$trip_duration <- as.integer(as.duration(dropoff_datetime - pickup_datetime))
  
  data
}
```

Before we apply the transformation to the data, it's usually a good idea to test it and make sure it's working.  We set aside a sample of the data as a `data.frame` for this purpose. Running the transformation function on `nyc_sample_df` should return the original data with the new columns.

```R
library(lubridate)
Sys.setenv(TZ = "US/Eastern") # not important for this dataset
head(xforms(nyc_sample_df)) # test the function on a data.frame
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
  improvement_surcharge total_amount pickup_hour pickup_dow dropoff_hour
1                   0.3          8.8    10PM-1AM        Fri     10PM-1AM
2                   0.3         19.3    10PM-1AM        Fri     10PM-1AM
3                   0.3         34.3    10PM-1AM        Fri     10PM-1AM
4                   0.3         17.3    10PM-1AM        Fri     10PM-1AM
5                   0.3          8.8    10PM-1AM        Fri     10PM-1AM
6                   0.3         20.3    10PM-1AM        Fri     10PM-1AM
  dropoff_dow trip_duration
1         Fri             0
2         Fri             0
3         Fri             0
4         Fri             0
5         Fri             0
6         Fri          1110
```

We run one last test before applying the transformation.  Recall that `rxDataStep` works with a `data.frame` input too, and that leaving the `outFile` argument means we return a `data.frame`.  So we can perform the above test with `rxDataStep` by passing transformation function to `transformFunc` and specifying the required packages in `transformPackages`.

```R
head(rxDataStep(nyc_sample_df, transformFunc = xforms, transformPackages = "lubridate"))
```

```Rout
Rows Processed: 1000 
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
  improvement_surcharge total_amount pickup_hour pickup_dow dropoff_hour
1                   0.3          8.8    10PM-1AM        Fri     10PM-1AM
2                   0.3         19.3    10PM-1AM        Fri     10PM-1AM
3                   0.3         34.3    10PM-1AM        Fri     10PM-1AM
4                   0.3         17.3    10PM-1AM        Fri     10PM-1AM
5                   0.3          8.8    10PM-1AM        Fri     10PM-1AM
6                   0.3         20.3    10PM-1AM        Fri     10PM-1AM
  dropoff_dow trip_duration
1         Fri             0
2         Fri             0
3         Fri             0
4         Fri             0
5         Fri             0
6         Fri          1110
```

Everything seems to be working well.  This does not guarantee that running the transformation function on the whole dataset will succeed, but it makes it less likely to fail for the wrong reasons.  If the transformation works on the sample `data.frame`, as it does above, but fails when we run it on the whole dataset, then it is usually because of something in the dataset that causes it to fail (such as missing values) that was not present in the sample data.  We now run the transformation on the whole data set.

```R
st <- Sys.time()
rxDataStep(nyc_xdf, nyc_xdf, overwrite = TRUE, transformFunc = xforms, transformPackages = "lubridate")
Sys.time() - st
```

```Rout
Time difference of 11.07041 mins
```
