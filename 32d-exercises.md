---
layout: page
title: Overview of RevoScaleR
---
# Exercises

In this exercise, we will be using the `nyc_jan_xdf` data from prior exercises. We also add `card_vs_cash` and `tip_percent`, `pickup_dow`, `pickup_hour`, and `trip_duration` as new columns to the data. If you need to re-load the data, run the following code:


```R
input_csv <- 'yellow_tripsample_2016-01.csv'
input_xdf <- 'yellow_tripsample_2016-01.xdf'
rxImport(input_csv, input_xdf, overwrite = TRUE)

nyc_jan_xdf <- RxXdfData(input_xdf)

rxDataStep(nyc_jan_xdf, nyc_jan_xdf, 
           transforms = list(
             card_vs_cash = factor(payment_type, levels = 1:2, labels = c('card', 'cash')),
             tip_percent = ifelse(tip_amount < fare_amount & fare_amount > 0, tip_amount / fare_amount, NA)
           ),
           overwrite = TRUE)

xforms <- function(data) { # transformation function for extracting some date and time features
  weekday_labels <- c('Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat')
  cut_levels <- c(1, 5, 9, 12, 16, 18, 22)
  hour_labels <- c('1AM-5AM', '5AM-9AM', '9AM-12PM', '12PM-4PM', '4PM-6PM', '6PM-10PM', '10PM-1AM')
  
  pickup_datetime <- ymd_hms(data$tpep_pickup_datetime, tz = "UTC")
  pickup_hour <- addNA(cut(hour(pickup_datetime), cut_levels))
  pickup_dow <- factor(wday(pickup_datetime), levels = 1:7, labels = weekday_labels)
  levels(pickup_hour) <- hour_labels
  
  dropoff_datetime <- ymd_hms(data$tpep_dropoff_datetime, tz = "UTC")
  
  data$pickup_hour <- pickup_hour
  data$pickup_dow <- pickup_dow
  data$trip_duration <- as.integer(as.duration(dropoff_datetime - pickup_datetime))
  
  data
}

rxDataStep(nyc_jan_xdf, nyc_jan_xdf, overwrite = TRUE, transformFunc = xforms, transformPackages = "lubridate")
```

(1) Build a linear model for predicting `tip_percent` using `trip_duration` and the interaction of `pickup_dow` and `pickup_hour`. Find out what your adjusted R-squared is by passing the model object to the `summary` function.

```R
formula_1 <- ## your formula goes here
linmod_1 <- ## build a linear model based on the above formula
```

Let's now try to improve our predictions by creating a better model. To do so, we can think of selecting "better" algorithms, but "better" is usually subjective as we discussed since every algorithm has its pros and cons and choosing between two algorithm can be a balancing act. However, one thing that any model can benefit from is better features. Better features can mean features that have been pre-processed to suit a particular algorithm, or it can refer to using more inputs in the model.

(2) Let's continue with linear models. Let's build a linear model very similar to the one represented by `formula_1`, except that we add `card_vs_cash` as input (a main effect). Let's call the new formula `formula_2`.

```R
formula_2 <- ## formula described above goes here
linmod_2 <- ## build a linear model based on the above formula
```

(3) Use `rxPredict` to put the predictions made by both models into the data as new columns called `tip_pred_1` and `tip_pred_2`. Then use `rxHistogram` to plot the predictions.

(4) It is also possible that are predictions are good, but need to be somewhat calibrated. To recalibrate the predictions, we use the `rescale` function in the `scales` library. In this case, let's rescale predictions so that both models predict a number between 0 and 25% tip. So use `rxDataStep` to write a transformation that does the following:

  - first replace predictions below 0 with `NA` and replace predictions above 25 with 25
  - then rescale the predictions to be between 0 and 25 using the `rescale` function (e.g. `rescale(x, to = c(0, 25))`)

```R
rxDataStep(nyc_jan_xdf, nyc_jan_xdf,
           transforms = list(
             ## transform tip_pred_1 and tip_pred_2 as described above
           ), overwrite = TRUE, transformPackages = "scales")
```

Now plot the histograms again and comment on the distribution of each plot. What does the bimodal (two separate concentration) shape of the distribution of `tip_pred_2` say about the model `linmod_2`?