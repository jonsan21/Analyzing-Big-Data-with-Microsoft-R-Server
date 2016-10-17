---
layout: default
title: Getting Started

---
# Getting started

Before we begin any analysis, we need to decide what tools we use. When performing **exploratory data analysis** R can be a great choice. It's syntax is succinct and very readable, and its plethora of packages can be used to our advantage. However, R is not without shortcomings. A `data.frame`, which is R's representation of tabular data, like any other R object needs to be loaded in the memory. When datasets get large, we quickly run out of available memory. Moreover, most analytics and modeling algorithms in R work only with a `data.frame`, but very large datasets are often stored in a distributed environment such as a Hadoop or Spark cluster or on a SQL Server. In such cases, we need algorithms that can work directly with data on disk or data distributed across a cluster, and we need algorithms that scale well as data sizes grow. In this module, we will see how the `RevoScaleR` package fills this need.