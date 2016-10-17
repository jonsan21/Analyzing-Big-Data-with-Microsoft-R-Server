# Clustering example

We now turn our attention to some of the analytics algorithms in `RevoScaleR`, starting with `rxKmeans` which is a parallel implementation of the k-means algorithm. Unlike the `kmeans` function, `rxKmeans` can run on an XDF file, a flat file, or data stored on HDFS or on SQL Server.

As we will see, when working with large datasets, tuning algorithms can have a great effect on performance.