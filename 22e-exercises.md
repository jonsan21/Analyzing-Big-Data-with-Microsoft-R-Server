# Exercises

In this exercise, we will be using the `nyc_jan_xdf` data from prior exercises. If you need to re-load the data, run the following code:

```R
input_csv <- 'yellow_tripsample_2016-01.csv'
input_xdf <- 'yellow_tripsample_2016-01.xdf'
rxImport(input_csv, input_xdf, overwrite = TRUE)

nyc_jan_xdf <- RxXdfData(input_xdf)
```

(1) Use `rxDataStep` along with the `rowSelection` argument to select the subset of rows with `trip_distance` greater than some threshold. The threshold is determined by a global variable called `dist_threshold` set below. Leave out the `outFile` argument so our result goes into a `data.frame` (which we call `nyc_long_trips_df`). We can hard-code this easily if the threshold is fixed, but letting a global variable decide the threshold makes the code more dynamic. Here's a hint: In order to pass a global R object to `rowSelection`, we need to use the `transformObjects` argument.

```R
dist_threshold <- 5 # a neighborhood of our choosing

nyc_long_trips_df <- rxDataStep(nyc_jan_xdf
	## you code goes here
	)
```

(2) How many rows do you have in the resulting subset `nyc_long_trips_df`?
