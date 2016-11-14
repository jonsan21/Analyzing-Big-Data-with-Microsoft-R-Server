# Exercises

In this exercise, we will be using the `nyc_jan_xdf` data from prior exercises. If you need to re-load the data, run the following code:

```R
input_csv <- 'yellow_tripsample_2016-01.csv'
input_xdf <- 'yellow_tripsample_2016-01.xdf'
rxImport(input_csv, input_xdf, overwrite = TRUE)

nyc_jan_xdf <- RxXdfData(input_xdf)
```

In the last section, we used the `kmeans` function to build clusters on the sample data, and used the centroids it provided us with to initialize the `rxKmeans` function, which builds the clusters on the whole data. One thing that we took for granted is the number of clusters to build. Our decision to build 300 clusters was somewhat arbitrary, based on the gut feeling that we expect about 300 "drop-off hubs" in Manhattan, i.e. points where taxis often drop passengers off. In this exercise, we provide a little more backing for our choice of the number of clusters.

Let's go back to the sample data and the `kmeans` function, as shown here. We made two changes to the function call:
  - we let the number of clusters vary based on what we pick for `nclus`
  - by letting `nstart = 1` we initialize the clusters only once, making it run much faster at the expense of less "stable" clusters (which we don't care about in this case)

```R
xydropoff <- rxDataStep(nyc_jan_xdf, rowSelection = (u < .1),
                     transforms = list(u = runif(.rxNumRows),
                                       long_std = dropoff_longitude / -74,
                                       lat_std = dropoff_latitude / 40),
                     varsToKeep = c("dropoff_longitude", "dropoff_latitude"))

xydropoff <- xydropoff[ , c("long_std", "lat_std")]

nclus <- 50
kmeans_nclus <- kmeans(xydropoff, centers = nclus, iter.max = 2000, nstart = 1)
sum(kmeans_nclus$withinss)
```

```Rout
[1] 0.0005410579
```

The number we extracted is the sum of the within-cluster sum of squares for each cluster. The **within-cluster sum of squares** (WSSs for short) is a measure of how much variability there is within each cluster. A lower WSSs indicates a more homogeneous cluster. However, we don't care about this metric per cluster. We simply sum over all the clusters' WSSs. When the number of clusters we build is small, individual clusters are less homogeneous, making the total WSSs larger. When we build a large number of clusters, the opposite is true. Therefore, total WSSs generally drops as `nclus` increases, but there is a point beyond which increasing `nclus` results in smaller and smaller drops in total WSSs. In other words, a point beyond which building a higher number of clusters is not worth the cost of increased complexity (as having more clusters makes it hard to tell them apart).

(1) Write a function called `find_wss` that has as input the number of clusters we want to build, represented by `nclus` in the above code, and returns the total WSSs. Additionally, your function should also print (but not return) the input `nclus` as well as how long it takes to run.

(2) Demonstrate the concept of decreasing total WSSs as we assign a larger and larger number to `nclus` by letting `nculs` loop through the increasing sequence of numbers given by `nclus_seq <- seq(20, 1000, by = 20)`, and running `find_wss` in each case. You can do so by plotting the results.