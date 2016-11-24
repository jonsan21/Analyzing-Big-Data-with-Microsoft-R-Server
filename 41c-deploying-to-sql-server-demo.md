---
layout: page
title: Overview of RevoScaleR
---
# Deploying to SQL Server (demo)

Let's point to a SQL table containing a copy of the NYC Taxi dataset. The first thing we need to do is set up a SQL Server *connection string*, which contains our SQL login credentials. Since the connection string contains sensitive information, it is usually stored in a file in a restricted location and read from R, but in our example we will simply hard-code the connection string and store it in `sqlConnString`. Assume, the NYC Taxi dataset is stored in a table called `NYCTaxiBig` inside the `RDB` database that the connection string points to. The last thing left for us to do is to point to the table, which we do with the `RxSqlServerData` function. This is the equivalent of `RxXdfData` when pointing to an XDF file stored on disk.

```R
sqlConnString <- "Driver=SQL Server;Server=SETHMOTTDSVM;Database=RDB;Uid=ruser;Pwd=ruser"
sqlRowsPerRead <- 100000
sqlTable <- "NYCTaxiBig"

nyc_sql <- RxSqlServerData(connectionString = sqlConnString, rowsPerRead = sqlRowsPerRead, table = sqlTable)
```

For the sake of illustration, we now dump the content of `nyc_xdf` into the SQL table represented by `nyc_sql` (which in called `NYCTaxiBig` in the SQL database). If the XDF file in question is large, this can take a while.

```R
system.time(
  rxDataStep(nyc_xdf, nyc_sql, overwrite = TRUE)
)
```

That's it. We can now use `nyc_sql` the same way we used `nyc_xdf` before. There is however something missing: we did not specify what the column types were. In this case, `RxSqlServerData` will try as best it can to convert a SQL Server column type to an R column type. This can cause problems though. First of all, SQL Server has a richer variety of column types than R. Second, some SQL Server column types like `datetime` for example don't always successfully transfer to their corresponding R column type. Third, the R column type `factor` does not really have a good equivalent in SQL Server, so in order for a column to be brought in as `factor` we must manually specify it. Doing so however gives us the advantage that we can also specify the levels and labels for it, and as we saw they don't always have to be the exact levels we see in the data. For example, if `payment_type` is represented by the integers 1 through 5 in the data, but we only care about 1 and 2 and want them labeled `card` and `cash` respectively, we can do that here without needing to do it later as a separate transformation. To deal with column types we create an object that stores the information about the columns and pass it to the `colInfo` argument in `RxSqlServerData`. Here's the example for `nyc_sql`:

```R
ccColInfo <- list(
  tpep_pickup_datetime = list(type = "character"),
  tpep_dropoff_datetime = list(type = "character"),
  passenger_count = list(type = "integer"),
  trip_distance = list(type = "numeric"),
  pickup_longitude = list(type = "numeric"),
  pickup_latitude = list(type = "numeric"),
  dropoff_longitude = list(type = "numeric"),
  dropoff_latitude = list(type = "numeric"),
  RateCodeID = list(type = "factor", levels = as.character(1:6), newLevels = c("standard", "JFK", "Newark", "Nassau or Westchester", "negotiated", "group ride")),
  store_and_fwd_flag = list(type = "factor", levels = c("Y", "N")),
  payment_type = list(type = "factor", levels = as.character(1:2), newLevels = c("card", "cash")),
  fare_amount = list(type = "numeric"),
  tip_amount = list(type = "numeric"),
  total_amount = list(type = "numeric")
)
```

Notice that the object above does not necessarily have to specify the types for each column in the data. We can limit it only to the columns of interest, and even then only the ones that need to be explicitly overwritten. However, since we tend to be more conservative in a production environment, it's best to be more explicit. After all, certain numeric columns in SQL Server could be stored as something different (`VARCHAR` for example) which would turn into a `character` column in R.

In addition to the columns that were in the original data, we also need to specify the column types for columns that we added to the data throughout the analysis. Let's begin with the date time columns:

```R
weekday_labels <- c('Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat')
hour_labels <- c('1AM-5AM', '5AM-9AM', '9AM-12PM', '12PM-4PM', '4PM-6PM', '6PM-10PM', '10PM-1AM')

ccColInfo$pickup_dow <- list(type = "factor", levels = weekday_labels)
ccColInfo$pickup_hour <- list(type = "factor", levels = hour_labels)
ccColInfo$dropoff_dow <- list(type = "factor", levels = weekday_labels)
ccColInfo$dropoff_hour <- list(type = "factor", levels = hour_labels)
```

When working with the XDF file in the previous weeks, we went back and forth quite a few times to get the data in the right format, especially where `factor` columns were concerned. This is because we were learning about the data as we went and gradually preparing it for analysis. When working in a production environment however, the assumption is that we have our EDA behind us and know quite a bit about the data already. If not, then a recommended approach would be to take a sample of the data first and run some EDA on it. So some of the steps that we took with the XDF file in the prior weeks may have contained some redundancy or inneficiencies, which we never bothered to go back and revise. But when deploying code in production it's a good idea to make a second pass at the code and simplify things wherever it's waranted. As an example, when working with the XDF file, we first wrote a function to extract the `pickup_nhood` and `dropoff_nhood` columns from the pick-up and drop-off coordinates. We then noticed that those columns contain neighborhoods outside of Manhattan limits (our area of interest), so we made a second pass through the data to remove the factor levels for the irrelevant neighborhoods. With `nyc_sql`, we could take a similar approach: read those columns as `factors` with levels as is, and then use `rxDataStep` to perform a transformation that removes unwanted factor levels. But doing so is inneficient. The better approach is to find all the relevant factor levels (Manhattan neighborhoods, which we can get directly from the shapefile) and in the `ccColInfo` object only specify those as levels for those columns. Here's how:

```R
nyc_shapefile <- readShapePoly('ZillowNeighborhoods-NY/ZillowNeighborhoods-NY.shp')
mht_shapefile <- subset(nyc_shapefile, str_detect(CITY, 'New York City-Manhattan'))
manhattan_nhoods <- as.character(mht_shapefile@data$NAME)

ccColInfo$pickup_nhood <- list(type = "factor", levels = manhattan_nhoods)
ccColInfo$dropoff_nhood <- list(type = "factor", levels = manhattan_nhoods)
```

We are now ready to point to the SQL table a second time, but this time specify how columns should be treated in R using the `colInfo` argument.

```
nyc_sql <- RxSqlServerData(connectionString = sqlConnString, table = sqlTable, rowsPerRead = sqlRowsPerRead, colInfo = ccColInfo)
```

Just recall that everytime we make a change to `ccColInfo`, we need to rerun the above line so that the change is reflected. For example, later (after running the `seriate` function), we can reorder the factor levels so that instead of being alphabetically ordered as they are now, they can follow a more natural ordering based on proximity to each other. 

At this point, the rest of the analysis is no different from what it was with the XDF file, so we can change `nyc_xdf` into `nyc_sql` and run the remaining code just like before. For example, we can start with `rxGetInfo` and `rxSummary` to double check the column types and get some summary statistics.

```R
rxGetInfo(nyc_sql, getVarInfo = TRUE, numRows = 3) # show column types and the first 10 rows

system.time(
  rxsum_sql <- rxSummary( ~ ., nyc_sql) # provide statistical summaries for all the columns
)
rxsum_sql
```

There are however some limitations that we need to be aware of. Certain functions, such as `rxMerge`, `rxSort` or `rxSplit` only work with XDF files on the local file system, not with data sitting in Spark or SQL Server. This is because the common data processing functions already have their (probably more efficient) implementation, so we can simply defer to the SQL language if we need to join tables, sort tables, or split tables instead of the above-mentioned `RevoScaleR` functions.

As an example, let's run the same linear model we build on the XDF file now using the SQL table. We're going to build the model on 75 percent of the data (the training data) by creating a column `u` of random uniform numbers and using `rowSelection` only picking rows where `u < .75`.

```R
system.time(linmod <- rxLinMod(tip_percent ~ pickup_nhood:dropoff_nhood + pickup_dow:pickup_hour, 
                               data = nyc_sql, reportProgress = 0,
                               rowSelection = (u < .75)))
```

We now point to a new SQL table called `NYCTaxiScore` (our pointer to it in R will be called `nyc_score`). We then use `rxPredict` to score the testing data (rows in `nyc_sql` where `u >= .75`) and dump the scores in it.

```R
sqlTable <- "NYCTaxiScore"
nyc_score <- RxSqlServerData(connectionString = sqlConnString, rowsPerRead = sqlRowsPerRead, 
                           table = sqlTable)

rxPredict(trained.models$linmod, data = nyc_sql, outData = nyc_score, predVarNames = "tip_percent_pred_linmod", overwrite = TRUE, rowSelection = (u >= .74))
```

While the above code does work, there is something very important missing from it from an architecture perspective. All of the computation so far has been happening in the client machine (the one hosting our current R session). Usually, the client machine (which could be our laptop) is not the same as the machine hosting SQL Server. This means that to run the computation, we had to transfer the data over to the client machine (using what's called an ODBC connection). If the SQL table in question is large, the data transfer will take a long time (how long depends on the underlying network infrastructure). Additionally, transferring data like this can put the data at risk. **In-database analytics** means that we go the other-way around: instead of bringing the data to the analytics, we take the analytics to the data. In other words, we run the analytics not the the current R session on our client machine, but by triggering a new (remote) R session on the machine hosting SQL Server. Doing so, we run the analytics remotely (and without having to transfer the data out of the SQL Server host) and only transfer the *results* to the client R session. This of course requires that the R installation on the SQL Server host be in sync with the R installation on the client machine, and that in particular any packages used in the code also be installed on it. 

All the above is usually handled by the SQL Server admin, not individual users. But in order to run in-database analytics, the R user has to do one thing from the R client machine, and that is to set the **compute context** to be the remote SQL/R server.


