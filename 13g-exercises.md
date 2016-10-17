# Exercises

The NYC Taxi dataset has a [data dictionary](http://www.nyc.gov/html/tlc/downloads/pdf/data_dictionary_trip_records_yellow.pdf), which we can use to properly label the columns `VendorID`, `RateCodeID`, `store_and_fwd_flag` and `payment_type`. 

(1) Based on the information in the data dictionary, run a transformation that coverts `RateCodeID` and `payment_type` into `factor` columns (if it's not one already) with the proper labels. In the case of `payment_type`, lump anything that isn't card or cash into a generic third category.

(2) What is the appropriate column types for `pickup_name` and `pickup_city`? Justify your answer. Check that the column types match your expectations.
