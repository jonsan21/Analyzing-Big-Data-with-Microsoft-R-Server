---
layout: page
title: Installing the Microsoft R Client
---

# Installing the Microsoft R Client

The `RevoScaleR` package is a Microsoft offering and it is **not** available on CRAN. To get `RevoScaleR` and its full functionality, we need a license for Microsoft R Server. However, we can get a free, light-weight version of Microsoft R Server by installing the [Microsoft R Client](https://msdn.microsoft.com/en-us/microsoft-r/r-client-get-started). Microsoft R Client gives us an installation of R which has the `RevoScaleR` library built into it, and loaded by default when we start an R session. Once we install it all we need to do is point our IDE (Visual Studio or RStudio) to the Microsoft R Client installation instead of any other installation of R (if there is no other installation of R, the IDE usually automatically detects the R Client installation). Using Microsoft R Client, we can develop code in R that leverages the `RevoScaleR` functions. This is a great way to learn how to use `RevoScaleR` and what functionality is offered. However, because Microsoft R Client is a free and light-weight version of Microsoft R Server, we are still bound by [some limitations](https://msdn.microsoft.com/en-us/microsoft-r/#mrc) mainly around data-size and performance. 

Please follow the steps in the link provided above to get Microsoft R Client installed. Additionally, we need an R IDE such as Visual Studio with [RTVS](https://www.visualstudio.com/vs/rtvs/) or RStudio.