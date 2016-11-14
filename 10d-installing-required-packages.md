---
layout: page
title: Overview of RevoScaleR
---
# Loading Packages

At various points throughout the analysis, we will be using to a set a third-party packages. So let's begin by loading those packages. The `RevoScaleR` package is pre-installed with Microsoft R Server (MRS), since it cannot be downloaded from CRAN, but all the other packages shown below are third-party packages that can be downloaded and installed from CRAN using the `install.packages` command.

Additionally, we override some default options to make it easier to display data or results.

```R
options(max.print = 1000, scipen = 999, width = 100)
library(RevoScaleR)
rxOptions(reportProgress = 1) # reduces the amount of output RevoScaleR produces
library(dplyr)
options(dplyr.print_max = 200)
options(dplyr.width = Inf) # shows all columns of a tbl_df object
library(stringr)
library(lubridate)
library(rgeos) # spatial package
library(sp) # spatial package
library(maptools) # spatial package
library(ggmap)
library(ggplot2)
library(gridExtra) # for putting plots side by side
library(ggrepel) # avoid text overlap in plots
library(tidyr)
library(seriation) # package for reordering a distance matrix
```