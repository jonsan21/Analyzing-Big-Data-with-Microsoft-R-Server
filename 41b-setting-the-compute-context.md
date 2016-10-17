# Setting up the compute context

The **compute context** refers to the environment in which the computation is happening. It is key to understanding how `RevoScaleR` achieves WODA. At a low level, the same computation will run differently in different compute context, but produce the same results.