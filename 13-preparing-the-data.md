# Preparing the data

Raw data is often too primitive to be used directly for analysis. After reading in the raw data, a data scientist spends a great deal of time and effort on cleaning the data and adding the features to the data that pertain to the analysis at hand. How the data needs to be cleaned is something that is partly guided by how the analysis makes business common sense and meets certain requirements, and partly by the specific analytics algorithm that it is being fed to. In other words, it is can be somewhat subjective as long as it does not makes the analysis hard to understand.

Common data prep tasks can be
  - dealing with missing values
  - dealing with outliers
  - deciding the level of granularity for the data, for example should time variables be in seconds, minutes, or hours, etc.
  - deciding which features to add or extract based on existing features to make the analysis more interesting or easier to interpret

### Learning objectives

After reading this chapter, we will understand how to
  - run some preliminary checks on the data
  - use `rxDataStep` modify existing columns or create new ones
  - wrap more complicated transformations into function that we pass directly to `rxDataStep`
  - examine and summarize new features to double-check our work

