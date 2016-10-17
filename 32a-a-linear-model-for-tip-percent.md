# A linear model predicting tip percent

Let's begin by building a linear model involving two interactive terms: one between `pickup_nb` and `dropoff_nb` and another one between `pickup_dow` and `pickup_hour`. The idea here is that we think trip percent is not just influenced by which neighborhood the passengers was pickup up from, or which neighborhood they were dropped off to, but which neighborhood they were picked up from AND dropped off to. Similarly, we intuit that the day of the week and the hour of the day together influence tipping. For example, just because people tip high on Sundays between 9 and 12, doesn't mean that they tend to tip high any day of the week between 9 and 12 PM, or any time of the day on a Sunday. This intuition is encoded in the model formula argument that we pass to the `rxLinMod` function: `tip_percent ~ pickup_nb:dropoff_nb + pickup_dow:pickup_hour` where we use `:` to separate interactive terms and `+` to separate additive terms.

```R
form_1 <- as.formula(tip_percent ~ pickup_nb:dropoff_nb + pickup_dow:pickup_hour)
rxlm_1 <- rxLinMod(form_1, data = mht_xdf, dropFirst = TRUE, covCoef = TRUE)
```    

Examining the model coefficients individually is a daunting task because of how many there are. Moreover, when working with big datasets, a lot of coefficients come out as statistically significant by virtue of large sample size, without necessarily being practically significant. Instead for now we just look at how our predictions are looking. We start by extracting each variable's factor levels into a `list` which we can pass to `expand.grid` to create a dataset with all the possible combinations of the factor levels. We then use `rxPredict` to predict `tip_percent` using the above model.


```R
rxs <- rxSummary( ~ pickup_nb + dropoff_nb + pickup_hour + pickup_dow, mht_xdf)
ll <- lapply(rxs$categorical, function(x) x[ , 1])
names(ll) <- c('pickup_nb', 'dropoff_nb', 'pickup_hour', 'pickup_dow')
pred_df_1 <- expand.grid(ll)
pred_df_1 <- rxPredict(rxlm_1, data = pred_df_1, computeStdErrors = TRUE, writeModelVars = TRUE)
names(pred_df_1)[1:2] <- paste(c('tip_pred', 'tip_stderr'), 1, sep = "_")
head(pred_df_1, 10)
```

```Rout
   tip_pred_1 tip_stderr_1          pickup_nb dropoff_nb pickup_dow pickup_hour
1    6.796323   0.16432197          Chinatown  Chinatown        Sun     1AM-5AM
2   10.741766   0.15853956       Little Italy  Chinatown        Sun     1AM-5AM
3    9.150114   0.09162002            Tribeca  Chinatown        Sun     1AM-5AM
4   10.174307   0.09819651               Soho  Chinatown        Sun     1AM-5AM
5    9.706202   0.07365164    Lower East Side  Chinatown        Sun     1AM-5AM
6    8.475197   0.06354026 Financial District  Chinatown        Sun     1AM-5AM
7   10.866035   0.07150005  Greenwich Village  Chinatown        Sun     1AM-5AM
8   10.997276   0.06831955       East Village  Chinatown        Sun     1AM-5AM
9    9.313165   0.12507373       Battery Park  Chinatown        Sun     1AM-5AM
10  10.613802   0.11624956       West Village  Chinatown        Sun     1AM-5AM
```
