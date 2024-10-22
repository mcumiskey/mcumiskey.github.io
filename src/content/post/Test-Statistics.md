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

Hypothesis tests are named after their test statistics because they’re the quantity that the tests actually evaluate. T-tests assess t-values, F-tests evaluate F-values, and chi-square tests assess chi-square values.

## Alpha threshold

The alpha threshold determines the amount of false positives you are willing to get. 

* By default, most people choose **0.05**. This is a debated number, so check your data. Healthcare and finance frequently use smaller values.

**Alpha Errors** / Type 1 Errors: *False Positives* 
* Reject the null when I should have failed to reject the null
* “I said x had a **different** mean weight than the mean population, but actually they are the **same** as the mean population.”

**Beta Errors** / Type 2 Errors: *False Negatives*
* I was in favor of the null when in fact I should have rejected the null. 
* “I said that x had the **same** weight as the mean population, but they actually have a **significantly different** mean.” 

# Z-Tests

### Requirements:
- population mean 
- standard deviation
- continuous data
- Your data follows a normal distribution
- - Or your sample is large enough to apply the [central limit theorem](https://statisticsbyjim.com/basics/central-limit-theorem/)
- If testing two samples, the groups must contain different sets of items. 

Since the Z-Test requires a *known* standard deviation, it is less common than a T-Test, which uses an *estimated* standard deviation.

A z-score for a specific data point (in a normal distribution) is simply the distance to the mean in the units of standard deviation. 

## Basic Formula for a one-sample Z-Test
We calculate z-scores with the following formula:
>  .
> $$ \large z = \frac{\bar{x} - \mu}{\sigma} $$
>  .

Where $\bar{x}$ is the sample mean, $\mu$ is the sample mean, and $\sigma$ is the size of the sample. 

```python
from scipy.stats import norm
import math
import pandas as pd 
#import the iris data set 
from sklearn.datasets import load_iris

# Set up the parameters
x = 7.4
mu = 8
n = 30
sigma = 5.4

# Calculate the z score for sample related to the population
z = (x - mu) / (sigma / math.sqrt(n))
print(f'The associated z score is {round(z, 4)}')
```

## Interpreting and using Z-scores

We now have our z-score! Wooooooooo!!!

- **Positive z-score:** The individual value is greater than the mean.
- **Negative z-score:** The individual value is less than the mean.
- **A z-score of 0:** The individual value is equal to the mean.

This does not give us a TON of useful information, so now we will use our z-score to find our P-score

```python
stats.norm.cdf(z-score)

left = norm.cdf(z)
right = norm.sf(z)

print(f'The probability on the left and right tails:\
      ({round(left, 4)}, {round(right, 4)})')
```
The associated z score is -0.6086
The probability on the left and right tails: (0.2714, 0.7286)

# T-Tests
1-Sample T-Test
- One sample compared to an unknown population 
- **OR** the sample you are trying to compare to the population is small (less than 30)

[ADD FORMULA]

2-Sample T-Test *(The most important T-Test)*
- Comparing sample 1 to sample 2
- **“Is the mean of the x sample significantly different than the mean of the y sample.”**

[ADD FORMULA]

[ADD NULL / ALTERNATIVE HYPOTHESIS]

[ADD CODE EXAMPLE]

# ANOVA Test (F One-Way Test)

ANOVA works by comparing the variance of **group means** to the **variance within groups.** This process determines if the groups are part of one larger population or separate populations with different means.
- It tests continuous data, not discrete. 

It allows us to test **3 or more groups at once**. 

The traditional F-test ANOVA assumes that **all groups have equal variances.**

Null Hypothesis: All of the group means are equal. 

Alternative Hypothesis: At least one mean is different. 


# Chi-Squared Tests

