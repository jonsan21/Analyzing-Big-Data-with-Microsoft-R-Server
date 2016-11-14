---
layout: page
title: Overview of RevoScaleR
---
# Filtering by Manhattan

We now narrow our field of vision by focusing on trips that took place inside of Manhattan only, and meet "reasonable" criteria for a trip.  Since we added new features to the data, we can also drop some old columns from the data so that the data can be processed faster.

```R
input_xdf <- 'yellow_tripdata_2016_manhattan.xdf'
mht_xdf <- RxXdfData(input_xdf)

rxDataStep(nyc_xdf, mht_xdf, 
           rowSelection = (
             passenger_count > 0 &
               trip_distance >= 0 & trip_distance < 30 &
               trip_duration > 0 & trip_duration < 60*60*24 &
               str_detect(pickup_borough, 'Manhattan') &
               str_detect(dropoff_borough, 'Manhattan') &
               !is.na(pickup_nb) &
               !is.na(dropoff_nb) &
               fare_amount > 0), 
           transformPackages = "stringr",
           varsToDrop = c('extra', 'mta_tax', 'improvement_surcharge', 'total_amount', 
                          'pickup_borough', 'dropoff_borough', 'pickup_nhood', 'dropoff_nhood'),
           overwrite = TRUE)
```

And since we limited the scope of the data, it might be a good idea to create a sample of the new data (as a `data.frame`).  Our last sample, `nyc_sample_df` was not a good sample, since we only took the top 1000 rows of the data.  This time, we use `rxDataStep` to create a random sample of the data, containing only 1 percent of the rows from the larger dataset.

```R
mht_sample_df <- rxDataStep(mht_xdf, rowSelection = (u < .01), 
                            transforms = list(u = runif(.rxNumRows)))

dim(mht_sample_df)
```

```Rout
Rows Processed: 57493035 
WARNING: The number of rows (574832) times the number of columns (24)
exceeds the 'maxRowsByCols' argument (3000000). Rows will be truncated.

> dim(mht_sample_df)
[1] 125000     24
```