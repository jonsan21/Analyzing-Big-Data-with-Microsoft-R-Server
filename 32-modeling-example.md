---
layout: page
title: Overview of RevoScaleR
---
# Modeling example

Trying to modeling a given behavior can be a very involved task, as the data itself and business requirements have a say into our choice for a model. Some models have higher predictive power but are less easy to interpret, and others are the other way around. Moreover, the process of building a model can also involves several stages such as choosing among many models then iterating so we can tune the model we've decided upon.

Our next exercise will consist of using several analytics functions offered by `RevoScaleR` to build models for predicting the amount customers tip for a trip. We will use the pick-up and drop-off neighborhoods and the time and day of the trip as the variables most likely to influence tip.

### Learning objectives

At the end of this chapter, we will have a better understanding of how
  - to build models with `RevoScaleR`
  - to understand trade-offs between various models
  - use visualizations to guide the process of choosing among certain models
  - to **score** (run predictions on) a dataset with a model we just built

This chapter is not necessarily a thorough guide on running and choosing models. Instead it offers  examples with code implementation as a starting point toward that end goal.