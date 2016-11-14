# Exercises

The NYC Taxi dataset has a [data dictionary](http://www.nyc.gov/html/tlc/downloads/pdf/data_dictionary_trip_records_yellow.pdf), which we can use to properly label the columns `VendorID`, `RateCodeID`, `store_and_fwd_flag` and `payment_type`. 

First check the column type for `payment_type`. Now based on the information in the data dictionary, run a transformation that creates a column called `card_vs_cash` (type `factor`) based `payment_type`. It will have two levels `card` and `cash` (lumping anything that isn't card or cash into an <NA> category).

We will be running the above transformation on `nyc_jan_xdf`, which is created here:

```R
input_csv <- 'yellow_tripsample_2016-01.csv'
input_xdf <- 'yellow_tripsample_2016-01.xdf'
rxImport(input_csv, input_xdf, overwrite = TRUE)

nyc_jan_xdf <- RxXdfData(input_xdf)
## your code goes here
```
