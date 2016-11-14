---
layout: page
title: Overview of RevoScaleR
---
# Reading the whole data

We now read the whole data using MRS. MRS has two ways of dealing with flat files:

1. it can work directly with the flat files, meaning that it can read and write to flat files directly, 
2. it can covert flat files to a format called XDF (XDF stands for **external data frame**).

We choose to go with the second option. We explain our reasoning in the next section. To convert flat files to XDF, we use the `rxImport` function. By letting `append = "rows"`, we can also combine multiple flat files into a single XDF file.

```R
input_xdf <- 'yellow_tripdata_2016.xdf'
library(lubridate)
most_recent_date <- ymd("2016-07-01") # the day of the months is irrelevant

st <- Sys.time()
for(ii in 1:6) { # get each month's data and append it to the first month's data
  file_date <- most_recent_date - months(ii)
  input_csv <- sprintf('yellow_tripsample_%s.csv', substr(file_date, 1, 7))
  append <- if (ii == 1) "none" else "rows"
  rxImport(input_csv, input_xdf, colClasses = col_classes, overwrite = TRUE, append = append)
  print(input_csv)
}
Sys.time() - st # stores the time it took to import
```

```Rout
Rows Processed: 10906858
[1] "yellow_tripdata_2016-01.csv"
Rows Processed: 11382049 
[1] "yellow_tripdata_2016-02.csv"
Rows Processed: 12210952 
[1] "yellow_tripdata_2016-03.csv"
Rows Processed: 11934338 
[1] "yellow_tripdata_2016-04.csv"
Rows Processed: 11836853 
[1] "yellow_tripdata_2016-05.csv"
Rows Processed: 11135470 
[1] "yellow_tripdata_2016-06.csv"
    
Time difference of 13.96592 mins
```
