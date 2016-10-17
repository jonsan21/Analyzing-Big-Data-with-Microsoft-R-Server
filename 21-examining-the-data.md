# Examining the data

In addition to asking whether the data makes logical sense, it's often a good idea to also check whether the data makes business sense or practical sense. Doing so can help us catch certain errors in the data such as data being mislabeled or attributed to the wrong set of features. If unaccounted, such soft errors can have a profound impact on the analysis.

### Learning objectives

After reading this chapter, we will understand how to
  - run basic tabulations and summaries on the data
  - take the return objects from `RevoScaleR` summary functions for further processing by R functions for plotting and more
  - visualize the distribution of a column with `rxHistogram`
  - take a random sample of the large data and use it as a way to examine outliers