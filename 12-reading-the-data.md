---
layout: page
title: Overview of RevoScaleR
---
# Reading the data

An analysis usually begins with a question we're trying to answer, after which we gather any data that can help us answer it. There are also times when we start with data we've collected and instead of trying to answer a specific question, we explore the data in search of not-so-obvious trends. This is sometimes referred to as **exploratory data analysis** and it can be a great way to help determine what sorts of questions the data can answer.

### Learning objectives

After reading this section we will understand
  - how `RevoScaleR` functions can work with data in the memory (`data.frame`) and with data on disk
  - data on disk can consist of flat files (such as CSV files), MRS's proprietary XDF format, and it can be stored locally or in a distributed file system such as HDFS
  - XDF files can be created from the original flat files using `rxImport`
  - the choice of converting from flat files to XDF depends on certain trade-offs
