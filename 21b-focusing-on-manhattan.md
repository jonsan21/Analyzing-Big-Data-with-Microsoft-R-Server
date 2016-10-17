# Focusing on Manhattan

Since the lion's share of taxi trips take place in Manhattan, we focus our attention to Manhattan only and ignore the other four boroughs.  For that purpose, we create two new columns called `pickup_nb` and `dropoff_nb` based on the original columns `pickup_nhood` and `dropoff_nhood` except that their factor levels are limited to Manhattan neighborhoods (any other factor level will be replaced with an NA).  It is important to do so, because otherwise neighborhoods outside of Manhattan will show up in any modeling or summary function involving those columns.

```R
manhattan_nhoods <- rownames(nhoods_by_borough)[nhoods_by_borough$`New York City-Manhattan` > 0]

refactor_columns <- function(dataList) {
  dataList$pickup_nb = factor(dataList$pickup_nhood, levels = nhoods_levels)
  dataList$dropoff_nb = factor(dataList$dropoff_nhood, levels = nhoods_levels)
  dataList
}

rxDataStep(nyc_xdf, nyc_xdf, 
           transformFunc = refactor_columns,
           transformObjects = list(nhoods_levels = manhattan_nhoods),
           overwrite = TRUE)

rxs_pickdrop <- rxSummary( ~ pickup_nb:dropoff_nb, nyc_xdf)
head(rxs_pickdrop$categorical[[1]])
```

```Rout
Rows Processed: 69406520 
 
      pickup_nb   dropoff_nb Counts
1  Battery Park Battery Park  19876
2 Carnegie Hill Battery Park   2699
3  Central Park Battery Park   3479
4       Chelsea Battery Park  61024
5     Chinatown Battery Park   3813
6       Clinton Battery Park  23962
```
