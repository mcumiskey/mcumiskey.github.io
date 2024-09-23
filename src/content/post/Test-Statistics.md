---
layout: ../../layouts/post.astro
title: Basic Statistical Tests In Python
description: Notes on Z-Tests, T-Tests, and Anova Testing
dateFormatted: Sept 19th, 2024
---

Imports:
```
import scipy 
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
from statsmodels.stats.power import GofChisquarePower
```

## Test Statistics 

The **test statistic** assesses how consistent sample data is with the null hypothesis in a hypothesis test.  

Hypothesis tests are named after their test statistics because they’re the quantity that the tests actually evaluate. T-tests assess t-values, F-tests evaluate F-values, and chi-square tests use chi-square values.

## Alpha threshold

The alpha threshold determines the amount of false positives you are willing to get. 

* By default, most people choose **0.05**. This is a debated number, so check your data. Healthcare and finance frequently use smaller values.

**Alpha Errors** / Type 1 Errors: *False Positives* 
* Reject the null when I should have failed to reject the null
* “I said x had a *different* mean weight than the mean population, but actually they are the *same* as the mean population.”

**Beta Errors** / Type 2 Errors: *False Negatives*
* I was in favor of the null when in fact I should have rejected the null. 
* “I said that x had the *same* weight as the mean population, but they actually have a *significantly different* mean.” 

```
def basicInterpretation(p_value, alpha):
    if p_value < alpha:
        return f"As our P-Value {p_value} is less than our alpha {alpha}, we can reject the null hypothesis."
    else:
        return f"As our P-Value {p_value} is greater than or equal to our alpha {alpha}, we fail to reject the null hypothesis."
```
# Z Tests

Used when we know the population mean, standard deviation, and our sample size is large enough (>30)

A z-score for a specific data point (in a normal distribution) is simply the distance to the mean in the units of standard deviations

```
# Import norm from scipy.stats
from scipy.stats import norm
import math

# Set up the parameters
x = 7.4
mu = 8
n = 30
sigma = 5.4

# Calculate the z score for sample related to the population
z = (x - mu) / (sigma / math.sqrt(n))
print(f'The associated z score is {round(z, 4)}')

# Calculate cumulative probability on the left tail
left = norm.cdf(z)

# Calculate the probability on the right tail
right = norm.sf(z)

print(f'The probability on the left and right tails:\
      ({round(left, 4)}, {round(right, 4)})')
```
The associated z score is -0.6086
The probability on the left and right tails:      (0.2714, 0.7286)