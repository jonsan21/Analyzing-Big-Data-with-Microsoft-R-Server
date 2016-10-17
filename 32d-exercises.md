# Exercises

In this section, we will try to improve our predictions. To do so, we can think of selecting "better" algorithms, but "better" is usually subjective as we discussed since every algorithm has its pros and cons and choosing between two algorithm can be a balancing act. However, one thing that any model can benefit from is better features. Better features can mean features that have been pre-processed to suit a particular algorithm, or it can refer to using more inputs in the model.

(1) Let's continue with linear models. Let's build a linear model very similar to the one represented by `form_1`, except that we throw `payment_type` into the mix. Let's call the new formula `form_3`.

```R
form_3 <- ## formula described above goes here
rxlm_3 <- ## build a linear model based on the above formula
```

We now create a dataset called `pred_df` which will contain all the combinations of the features contained in `form_3`.

```R
rxs <- rxSummary( ~ payment_type + pickup_nb + dropoff_nb + pickup_hour + pickup_dow, mht_xdf)
ll <- lapply(rxs$categorical, function(x) x[ , 1])
names(ll) <- c('payment_type', 'pickup_nb', 'dropoff_nb', 'pickup_hour', 'pickup_dow')
pred_df <- expand.grid(ll)
```

(2) Use `rxPredict` to put the predictions from the first model we built, `rxlm_1` and the current model, `rxlm_3` into this dataset. In other words, score `pred_df` with `rxlm_1` and `rxlm_3`. Dump results in a `data.frame` called `pred_all`. Name the predictions `p1` and `p3` respectively. Using the same binning process as before, replace the numeric predictions with the categorical predictions (e.g. `p1` is replaced by `cut(p1, c(-Inf, 8, 12, 15, 18, Inf))`). This is what the final dataset will look like:

```R
  payment_type    pickup_nb dropoff_nb pickup_hour pickup_dow      p1        p3
1         card    Chinatown  Chinatown     1AM-5AM        Sun  (8,12] (18, Inf]
2         cash    Chinatown  Chinatown     1AM-5AM        Sun  (8,12]  (-Inf,8]
3         card Little Italy  Chinatown     1AM-5AM        Sun (15,18] (18, Inf]
4         cash Little Italy  Chinatown     1AM-5AM        Sun (15,18]  (-Inf,8]
5         card      Tribeca  Chinatown     1AM-5AM        Sun (12,15] (18, Inf]
6         cash      Tribeca  Chinatown     1AM-5AM        Sun (12,15]  (-Inf,8]
```

We can see for example that for the first row `rxlm_1` predicted a tip between 8% and 12%, whereas `rxlm_3` predicted a tip of 18% or higher.

(3) Feed `pred_all` to the following code snippet to create a plot comparing the two predictions for different days and times of the day. What is your conclusion based on the plot?

```R
ggplot(data = pred_all) +
  geom_bar(aes(x = p1, fill = "model 1", group = payment_type, alpha = .5)) +
  geom_bar(aes(x = p3, fill = "model 3", group = payment_type, alpha = .5)) +
  facet_grid(pickup_hour ~ pickup_dow) +
  xlab('tip percent prediction') +
  theme(axis.text.x = element_text(angle = 90, hjust = 1))
```