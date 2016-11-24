---
layout: page
title: Overview of RevoScaleR
---
# Judging predictive performance

We now apply the model to the test data so we can compare the predictive power of each model. If we are correct about the decision tree over-fitting, then we should see it preform poorly on the test data compared to the other two models. If we believe the random forest captures some inherent signals in the data that the linear model misses, we should see it perform better than the linear model on the test data.

The first metric we look at is the average of the squared residuals, which gives us an idea of how close the predictions are to the observed values. Since we're predicting tip percent, which usually falls in a narrow range of 0 to about 20 percent, we should expect on average the residuals for a good model to be no more than 2 or 3 percentage points.

```R
rxPredict(trained.models$linmod, data = mht_split$test, outData = mht_split$test, predVarNames = "tip_percent_pred_linmod", overwrite = TRUE)
rxPredict(trained.models$dtree, data = mht_split$test, outData = mht_split$test, predVarNames = "tip_percent_pred_dtree", overwrite = TRUE)
rxPredict(trained.models$dforest, data = mht_split$test, outData = mht_split$test, predVarNames = "tip_percent_pred_dforest", overwrite = TRUE)

rxSummary(~ SE_linmod + SE_dtree + SE_dforest, data = mht_split$test,
          transforms = list(SE_linmod = (tip_percent - tip_percent_pred_linmod)^2,
                            SE_dtree = (tip_percent - tip_percent_pred_dtree)^2,
                            SE_dforest = (tip_percent - tip_percent_pred_dforest)^2))
```

```Rout
Call:
rxSummary(formula = ~ SE_linmod + SE_dtree + SE_dforest, data = mht_split$test, 
    transforms = list(SE_linmod = (tip_percent - tip_percent_pred_linmod)^2, 
        SE_dtree = (tip_percent - tip_percent_pred_dtree)^2, 
        SE_dforest = (tip_percent - tip_percent_pred_dforest)^2))

Summary Statistics Results for: ~SSE_linmod + SSE_dtree + SSE_dforest
Data: mht_split$test (RxXdfData Data Source)
File name: C:\Data\NYC_taxi\output\split\train.split.train.xdf
Number of valid observations: 43118543 
 
 Name        Mean     StdDev   Min                 Max      ValidObs MissingObs
 SE_linmod  82.66458 108.9904 0.00000000005739206 9034.665 43118542 1         
 SE_dtree   82.40040 109.1038 0.00000251589457986 8940.693 43118542 1         
 SE_dforest 82.47107 108.0416 0.00000000001590368 8606.201 43118542 1        
 ```

Another metric worth looking at is a correlation matrix. This can help us determine to what extent the predictions from the different models are close to each other, and to what extent each is close to the actual or observed tip percent.

```R
rxc <- rxCor( ~ tip_percent + tip_percent_pred_linmod + tip_percent_pred_dtree + tip_percent_pred_dforest, data = mht_split$test)
print(rxc)
```
 
```Rout
                          tip_percent pred_linmod pred_dtree pred_dforest
tip_percent               1.0000000   0.1391751   0.1500126  0.1499031
tip_percent_pred_linmod   0.1391751   1.0000000   0.8580617  0.9084119
tip_percent_pred_dtree    0.1500126   0.8580617   1.0000000  0.9404640
tip_percent_pred_dforest  0.1499031   0.9084119   0.9404640  1.0000000
```