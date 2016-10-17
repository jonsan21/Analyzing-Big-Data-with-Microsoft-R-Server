
# Visualizing the data

Visualizing data is a useful in order to get a good sense of the data. It can be a good way to anticipate what needs to be done to get the data ready for analysis. It can also be a good way to present our results without having to look at too many numbers.

For most visualizations, large datasets can present certain challenges. For example, having too many data points on a scatter plot can make the scatter plot too crowded to see any underlying trend. Too many points can also make it too long to process the visualization. Some other visualizations on the other hard, such as histograms or a bar plot, do not necessarily take longer to process with more data. This is why we have an `rxHistogram` function in `RevoScaleR` but not necessarily a function for creating scatter plots.

Therefore, when dealing with large datasets, it is far more common to create visualizations on aggregated data instead of the raw data. We will see several examples of this course. Once data is aggregated, with a little bit of reshaping we can prepare it for tools such as `ggplot2` to display the results.