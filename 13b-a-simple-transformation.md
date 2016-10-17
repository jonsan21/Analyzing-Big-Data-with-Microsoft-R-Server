# A simple transformation

Once data in brought in for analysis, we can begin thinking about the interesting\/relevant features that go into the analysis.  Our goal is primarily exploratory: we want to tell a story based on the data.  In that sense, any piece of information contained in the data can be useful.  Additionally, new information \(or features\) can be extracted from existing data points.  It is not only important to think of which features to extract, but also what their column type must be, so that later analyses run appropriately.  As a first example, consider a simple transformation for extracting the percentage that passengers tipped for the trip.

This is where we encounter the `rxDataStep` function, a function that we will revisit many times. `rxDataStep` is an essential function in that it is the most important data transformation function in `RevoScaleR` \(the others being `rxMerge` and `rxSort`\); most other analytics functions are data summary and modeling functions.  `rxDataStep` can be used to

* modify existing columns or add new columns to the data
* keep or drop certain columns from the data before writing to a new file
* keep or drop certain rows of the data before writing to a new file

In a local compute context, when we run `rxDataStep`, we specify an `inData` argument which can point to a `data.frame` or a CSV or XDF file. \(We can later change the compute context to SQL Server or HDFS to run the transformation in a remote, distributed compute context.\) We also have an `outFile` argument that points to the file we are outputting to, and if both `inData` and `outFile` point to the same file, we must set `overwrite = TRUE`.  **Note that **`outFile`** is an optional argument: leaving it out will output the result into a **`data.frame`**.  However, in most cases that is not what we want, so we need to specify **`outFile`**.**

Let's start with a simple transformation for calculating tip percentage.

```R
rxDataStep(nyc_xdf, nyc_xdf, 
           transforms = list(tip_percent = ifelse(fare_amount > 0 & tip_amount < fare_amount, round(tip_amount*100 / fare_amount, 0), NA)),
           overwrite = TRUE)
rxSummary( ~ tip_percent, nyc_xdf)
```

```Rout
Rows Processed: 69406520 

Call:
rxSummary(formula = ~tip_percent, data = nyc_xdf)

Summary Statistics Results for: ~tip_percent
Data: nyc_xdf (RxXdfData Data Source)
File name: yellow_tripdata_2016.xdf
Number of valid observations: 70710614 
 
 Name        Mean     StdDev   Min Max ValidObs MissingObs
 tip_percent 13.97823 11.87074 -1  100 70596806 113808  
 ```

There is a slightly different way that to get the above summary. We can perform the transformation directly inside `rxSummary` instead of in a prior `rxDataStep` statement. In this second way, the transformation is happening on-the-fly by `rxSummary` without being written to the data. Because of the lower IO overhead, this second method is more efficient *for a single run*. All the summary function and analytics functions in `RevoScaleR` allow us to create new columns on-the-fly like the following:

```R
rxSummary( ~ tip_percent2, nyc_xdf, 
  transforms = list(tip_percent2 = ifelse(fare_amount > 0 & tip_amount < fare_amount, round(tip_amount * 100 / fare_amount, 0), NA)))
```

```Rout
Rows Processed: 69406520 

Call:
rxSummary(formula = ~tip_percent, data = nyc_xdf)

Summary Statistics Results for: ~tip_percent
Data: nyc_xdf (RxXdfData Data Source)
File name: yellow_tripdata_2016.xdf
Number of valid observations: 70710614 
 
 Name        Mean     StdDev   Min Max ValidObs MissingObs
 tip_percent 13.97823 11.87074 -1  100 70596806 113808  
 ```

This is especially helpful if the transformations are straightforward transformations that are derived from existing columns in the data. In the following example, we run a transformation within `rxCrossTabs` to derive the month and year of the data from `tpep_pickup_datetime` (stored for now as a `character` column). We then convert them into `factor` columns so that `rxCrossTabs` can give us counts for each year and month combination.

```R
rxCrossTabs( ~ month:year, nyc_xdf, 
             transforms = list(
               date = ymd_hms(tpep_pickup_datetime), 
               year = factor(year(date), levels = 2014:2016), 
               month = factor(month(date), levels = 1:12)), 
             transformPackages = "lubridate")
```

```Rout
Call:
rxCrossTabs(formula = ~month:year, data = nyc_xdf, rowSelection = (year == 
    2016), transforms = list(date = ymd_hms(tpep_pickup_datetime), 
    year = factor(year(date), levels = 2014:2016), month = factor(month(date), 
        levels = 1:12)), transformPackages = "lubridate")

Cross Tabulation Results for: ~month:year
Data: nyc_xdf (RxXdfData Data Source)
File name: yellow_tripdata_2016.xdf
Number of valid observations: 58499662
Number of missing observations: 0 
Statistic: counts 
 
month:year (counts):
     year
month 2014 2015     2016
   1     0    0        0
   2     0    0 11382049
   3     0    0 12210952
   4     0    0 11934338
   5     0    0 11836853
   6     0    0 11135470
   7     0    0        0
   8     0    0        0
   9     0    0        0
   10    0    0        0
   11    0    0        0
   12    0    0        0
 ```