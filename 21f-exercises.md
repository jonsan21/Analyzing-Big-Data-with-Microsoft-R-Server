# Exercises

Let's re-create the histogram for `trip_distance` using `rxHistogram`:

```R
rxHistogram( ~ trip_distance, nyc_xdf, startVal = 0, endVal = 25, histType = "Percent", numBreaks = 20)
```

(1) Modify the formula in the line above so that we get a separate histogram for each combination of `pickup_hour` and `payment_type`.

We used `rxHistogram` to get a histogram of `trip_distance`, which is a `numeric` column. We can also feed a `factor` column to `rxHistogram` and the result is a bar plot. If a `numeric` column is heavily skewed, its histogram is often hard to look at because most of the information is squeezed to one side of the plot. In such cases, we can convert the `numeric` column into a `factor` column, a process that's also called **binning**. We do that in R using the `cut` function, and provide it with a set of breakpoints that are used as boundaries for moving from one bin to the next.

For example, let's say we wanted to know if taxi trips travel zero miles (for whatever business reason), 5 miles or less, between 5 and 10 miles, or 10 or more miles. If a taxi trip travels 8.9 miles, then the following code example will answer it for us.

```R
cut(8.9, breaks = c(-Inf, 0, 5, 10, Inf), labels = c("0", "<5", "5-10", "10+"))
```

```Rout
[1] 5-10
Levels: 0 <5 5-10 10+
```

(2) Modify the `rxHistogram` call in part (1) so that instead of plotting a histogram of `trip_distance`, you plot a bar plot of the trip distance binned based on the breakpoints provided in the above code example.