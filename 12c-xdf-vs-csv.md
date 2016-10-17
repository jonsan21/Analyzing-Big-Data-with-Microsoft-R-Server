# XDF vs CSV

An XDF file is much smaller than a CSV file because it is compressed. Its main advantage over a CSV file is that an XDF file can be read and processed much faster than a CSV file (we will run a simple benchmark to see how much faster). The disadvantage of an XDF file format is a format that only MRS understands and can work with. So in order to decide whether we chose XDF or CSV we need to understand the I/O trade-offs involved:

1. Converting from CSV to XDF is itself a cost in terms of runtime.
2. Once the original CSVs are converted to XDFs, the runtime of processing (reading from and sometimes writing to) the XDFs is lower than what the it would have been if we had directly processed the CSVs instead.

Since an EDA workflow usually consists of cleaning and munging data, and then feeding that to various modeling and data-mining algorithms, the initial runtime of converting from CSV to XDF is quickly offset by the reduced runtime of subsequently working with the XDF file. However, one-off kinds of analyses on datasets that are ready to be fed to the modeling algorithm might run faster if we skip XDF conversion. One-off operations are also common in production code, such as when a dataset is scored with an already existing model every time new data comes in. In such cases, we need to run some benchmarks in order to find the optimal solution.

In the last section, we used `rxImport` to covert 6 months worth of CSV files into a single XDF file.  We can now create an R object called `nyc_xdf` that points to this XDF data. We do so by providing the path to the XDF file to the `RxXdfData` function. Let's look at a summary of this dataset by running the `rxSummary` function against `nyc_xdf`. The `rxSummary` function uses the popular **formula notation** used by many R functions. In this case, the formula `~ fare_amount` means we want to see a summary for the `fare_amount` column only. Just like the `summary` function in `base` R, `rxSummary` will show us a different output depending on the type of each column. Since `fare_amount` is a numeric column, we get numeric summary statistics.

```R
input_xdf <- 'yellow_tripdata_2016.xdf'
nyc_xdf <- RxXdfData(input_xdf)
system.time(
rxsum_xdf <- rxSummary( ~ fare_amount, nyc_xdf) # provide statistical summaries for fare amount
)
rxsum_xdf
```

```Rout
Rows Processed: 69406520 
   user  system elapsed 
   0.00    0.00    1.68 

Call:
rxSummary(formula = ~fare_amount, data = nyc_xdf)

Summary Statistics Results for: ~fare_amount
Data: nyc_xdf (RxXdfData Data Source)
File name: yellow_tripdata_2016.xdf
Number of valid observations: 69406520 
 
 Name        Mean     StdDev   Min    Max      ValidObs MissingObs
 fare_amount 12.91626 128.1172 -957.6 628544.7 69406520 0         
```

Note that we could have done the same analysis with the original CSV file and skipped XDF conversion. Since we have a separate CSV file for each month, unless we combine the CSV files, we can only get the summary for one month's data.  For our purposes that will be enough. To run `rxSummary` on the CSV file, we simply create a pointer to the CSV file using `RxTextData` (instead of `RxXdfData` as was the case with the XDF file) and pass the column types directly to it using the `colClasses` argument. The rest is the same.  Notice how running the summary on the CSV file takes considerably longer (even though the CSV file comprises only one month's data).

```R
input_csv <- 'yellow_tripdata_2016-01.csv' # we can only use one month's data unless we join the CSVs
nyc_csv <- RxTextData(input_csv, colClasses = col_classes) # point to CSV file and provide column info
system.time(
  rxsum_csv <- rxSummary( ~ fare_amount, nyc_csv) # provide statistical summaries for fare amount
)
rxsum_csv
```

```Rout
Rows Processed: 10906858 
   user  system elapsed 
   0.00    0.00   42.58 

Call:
rxSummary(formula = ~fare_amount, data = nyc_csv)

Summary Statistics Results for: ~fare_amount
Data: nyc_csv (RxTextData Data Source)
File name: yellow_tripdata_2016-01.csv
Number of valid observations: 10906858 
 
 Name        Mean     StdDev Min    Max      ValidObs MissingObs
 fare_amount 12.48693 35.564 -957.6 111270.9 10906858 0         
```

The last example was run to demonstrate `RevoScaleR`'s capability to work directly with flat files (even though they take longer than XDF files), but since our analysis involves lots of data processing and running various analytics functions, from now on we work with the XDF file, so we can benefit from faster runtime.