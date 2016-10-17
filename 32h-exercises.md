# Exercises

The models we built in the last section were disappointing, and the last exercise showed us that it's because those models ignored `payment_type`, which is strongly correlated with `tip_percent` since customers who pay cash for their taxi fare do not have their tips registered by the data. 

(1) Now that we  used `RevoScaleR` to split the data into training and testing sets, let's go back and rebuild the three models from last section but include `payment_type` this time, in addition to all the features we used last time.

```R
linmod <- rxLinMod(## formula goes here
                   mht_split$train, reportProgress = 0)
dtree <- rxDTree(## formula goes here
                 mht_split$train, pruneCp = "auto", reportProgress = 0))
dforest <- rxDForest(## formula goes here
                     mht_split$train, nTree = 10, importance = TRUE, useSparseCube = TRUE, reportProgress = 0))

trained.models <- list(linmod = linmod, dtree = dtree, dforest = dforest)
save(trained.models, file = 'trained_models_2.Rdata')
```

(2) Here is a code to build a prediction dataset called `pred_df`. Fill in the code for predicting `payment_type` using the three models from above and attach the predictions to `pred_df` as separate columns called `pred_linmod`, `pred_dtree` and `pred_dforest`.

```R
rxs <- rxSummary( ~ payment_type + pickup_nb + dropoff_nb + pickup_hour + pickup_dow, mht_xdf)
ll <- lapply(rxs$categorical, function(x) x[ , 1])
names(ll) <- c('payment_type', 'pickup_nb', 'dropoff_nb', 'pickup_hour', 'pickup_dow')
pred_df <- expand.grid(ll)
## predict payment_type on pred_df by each of three models
```

(3) Run your dataset against the following code snippet to produce a plot similar to the plot we examined earlier. What is different about the plot this time?

```R
ggplot(data = pred_df) +
  geom_density(aes(x = pred_linmod, col = "linmod")) +
  geom_density(aes(x = pred_dtree, col = "dtree")) +
  geom_density(aes(x = pred_dforest, col = "dforest")) +
  xlab("tip percent")
```

(4) The `rxXdfToDataFrame` function dumps the content of an XDF file into a `data.frame` called `test_df`. We will use it to extract only the necessary columns from the test data and run predictions on it. In the last section, we put the predictions directly in the test data's XDF file, but because this is an IO intensive operation, we can run it faster by using a `data.frame` instead (as long as the test data is not big enough to make us run out of memory). Use `rxPredict` to run score `test_df` with the predictions from each of the three models.

```R
test_df <- rxXdfToDataFrame(mht_split$test, varsToKeep = c('tip_percent', 'payment_type', 'pickup_nb', 'dropoff_nb', 'pickup_hour', 'pickup_dow'), maxRowsByCols = 10^9)
test_df_1 <- ## predict for the first model, call the predictions `tip_pred_linmod`
test_df_2 <- ## predict for the second model, call the predictions `tip_pred_dtree`
test_df_3 <- ## predict for the third model, call the predictions `tip_pred_dforest`
```

We can now attach models' predictions to the `test_df` and run a `rxSummary` (or any R function we like, since `test_df` is a `data.frame`) to look at the performance of the models.

```R
test_df <- do.call(cbind, list(test_df, test_df_1, test_df_2, test_df_3))

rxSummary(~ SSE_linmod + SSE_dtree + SSE_dforest, data = test_df,
          transforms = list(SSE_linmod = (tip_percent - tip_pred_linmod)^2,
                            SSE_dtree = (tip_percent - tip_pred_dtree)^2,
                            SSE_dforest = (tip_percent - tip_pred_dforest)^2))
```

(5) Run the above summary and report your findings.