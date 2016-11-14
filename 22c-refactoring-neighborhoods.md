---
layout: page
title: Overview of RevoScaleR
---
# Refactoring neighborhoods

As the plots show, a lot of traffic happens between neighborhoods that are close to each other.  This is not very surprising since trips between neighborhoods that are far apart can be made using periphery routes that bypass most of the traffic through the town center.  We can also see generally high traffic in the midtown neighborhoods, and downtown especially between Chinatown and Little Italy.

We changed the order of the factor levels for `pickup_nb` and `dropoff_nb` to draw the above plots. However, this change best belongs in the data itself, otherwise every time we plot something involving `pickup_nb` or `dropoff_nb` we will need to change the order of the factor levels. So let's take the change and apply it to the whole data. We have two options for making the change:

  1. We can use `rxDataStep` with the `transforms` argument, and use the `base` R function `factor` to reorder the factor levels.
  2. We can use the `rxFactor` function and its `factorInfo` to manipulate the factors levels. The advantage of `rxFactors` is that it is faster, because it works at the meta-data level. The disadvantage is that it may not work in other compute contexts such as Hadoop or Spark.
  
Both ways of doing this are shown here.

```R
# first way of reordering the factor levels
rxDataStep(inData = mht_xdf, outFile = mht_xdf,
           transforms = list(pickup_nb = factor(pickup_nb, levels = newlevels),
                             dropoff_nb = factor(dropoff_nb, levels = newlevels)),
           transformObjects = list(newlevels = unique(newlevs)),
           overwrite = TRUE)
```


```R
# second way of reordering the factor levels
rxFactors(mht_xdf, outFile = mht_xdf, factorInfo = list(pickup_nb = list(newLevels = unique(newlevs)), 
                                                        dropoff_nb = list(newLevels = unique(newlevs))),
         overwrite = TRUE)
```
