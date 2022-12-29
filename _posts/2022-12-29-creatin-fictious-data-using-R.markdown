---
layout: post
title:  "Creating Fictitious Data for Factor Analysis Using the psych Package in R"
date:   2022-12-29 20:58:00 +0900
tags:
- R
---

Originally posted on 06/11/2021 to [qiita.com](https://qiita.com/sbtseiji/items/df91cb4667af3ab133d2)

I used to spend a lot of time creating fictitious data for my statistics classes, but now I have learned that it is quite easy to do so using the `psych` package in R. 

All you need is the `psych` package, and the `MASS` package. `sim.structure()` in the psych package creates a correlation matrix for the observed variables by specifying the factor loadings and inter-factor correlations, and `mvrnorm()` in the `MASS` package generates multivariate normal normally distributed random number sequence from them.

Then scale them appropriately to integers or n-point scale responses to complete the process. For creating n-point scale values, the `psych` package's `con2cat()` is helpful. If the sample size is small, the factor analysis results may not be as expected, but there's nothing you can do about it.

The procedure and code for creating fictitious data for factor analysis are as follows.

{% highlight R %}
require(psych)
require(MASS)

set.seed(20210611)

fload_mat <- matrix( # Specify the factor loading matrix
                    c( 0.8,  0.0,  0.0,
                      -0.7,  0.0,  0.0,
                       0.6,  0.0,  0.0,
                       0.0,  0.7,  0.1, 
                       0.0,  0.7,  0.1,
                       0.0,  0.0,  0.6,
                       0.0,  0.0,  0.6), ncol=3, byrow=T)

factor_cor <- matrix( # Specify the correlation matrix between factors
                      c(1.0, 0.4, 0.4,
                        0.4, 1.0, 0.2,
                        0.4, 0.2, 1.0), ncol=3)

cor_mat <- sim.structure( fload_mat , factor_cor )$model 

N <- 100

# Here the average value of the seven variables is set to all 0
var_means <- rep(0, 7)

rand_data <- mvrnorm( N , var_means , cor_mat )

#### Converting data to continuous variables (here converting to integer values with a mean score of around 50 with a standard deviation of 4 to 6)

# Scaling standard deviation from 4x to 6x
int_data <- rand_data * runif( n = 7, min = 4, max = 7)

# Changing the average to a range of 45 to 55
int_data <- int_data + runif( n = 7, min = 45, max = 55)

# make them integer
int_data <- round(int_data)

#### When creating fictious data as responses on a 7-point scale (from 1 to 7)

# Scaling standard deviation from 1.5x to 2x
res_data <- rand_data * runif( n = 7, min = 1.5, max = 2.0)

# Changing the average to between 2.5 to 3.5
res_data <- res_data + runif( n = 7, min = 2.5, max = 3.5)

# Cut at appropriate point to 7 levels and add minimum rating value
res_data <- con2cat(res_data,cuts=c( 0.5, 1.5, 2.5, 3.5, 4.5, 5.5)) + 1

{% endhighlight %}