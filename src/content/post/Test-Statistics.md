---
layout: ../../layouts/post.astro
title: Basic Statistical Tests In Python
description: Notes on Z-Tests, T-Tests, and Anova Testing
dateFormatted: Sept 19th, 2024
---

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

**Null hypothesis:** The population mean is **equal** to a specific value. 

**Alternative hypothesis**: The population mean is **different** from the known value (two-tailed), or it is greater than or less than the known value (one-tailed).

```python
from scipy.stats import norm
import math
import pandas as pd 

# Set up the parameters
x = 7.4
mu = 8
n = 30
sigma = 5.4

# Calculate the z score for sample related to the population
z = (x - mu) / (sigma / math.sqrt(n))
print(f'The associated z score is {round(z, 4)}')
```
```
The associated z score is -0.6086
```

A z-score of -0.6086 means the value is 0.6086 standard deviations below the mean of the distribution. In simpler terms, **this data point lies a bit less than 0.61 standard deviations to the left of the average** value of the distribution.


## Interpreting and using Z-scores

We now have our z-score! Wooooooooo!!!

- **Positive z-score:** The individual value is greater than the mean.
- **Negative z-score:** The individual value is less than the mean.
- **A z-score of 0:** The individual value is equal to the mean.

This does not give us a TON of useful information, so now we will use our z-score to find our **P-value**.

## P-Values

The p-value is a **probability** measure used in hypothesis testing to determine the significance of results. It represents the likelihood of obtaining a result at least as extreme as the one observed, given that the null hypothesis is true.

**Low p-value (≤ 0.05):** Suggests strong evidence against the null hypothesis, meaning the result is statistically significant.

**High p-value (> 0.05)**: Suggests weak evidence against the null hypothesis, meaning the result is not statistically significant.

For example, if you were testing whether a new drug is effective, a p-value of 0.03 would suggest that there is only a 3% chance that the observed effect was due to random chance, thus providing evidence that the drug has an effect.

```python
#Use stats .cdf (Cumulative Density Function)
p-score = stats.norm.cdf(z-score)

left = stats.norm.cdf(z)
# Can also be calculated as 1 - stats.norm.cdf(z)
right = stats.norm.sf(z)

print(f'The probability on the left and right tails:\
      ({round(left, 4)}, {round(right, 4)})')
```

```python
The probability on the left and right tails: 0.2714, 0.7286
```

This means the p-value (or the **probability** associated with this z-score) is approximately 0.2714. This tells us that about **27.14% of the values in a standard normal distribution are less than -0.6086.**

**Tails** are used when the alternative hypothesis contains something "greater than" or "less than", which means that we are looking at the area under the curve in two places, one on each side. **Our normal p-score is the left tail!**


# T-Tests
## 1-Sample T-Tests 
### Requirements:
- One sample compared to an unknown population 
- **OR** the sample you are trying to compare to a small population (less than 30)
>  .
> $$\large t = \frac{\bar{x}-\mu}{\frac{s}{\sqrt{n}}}$$
>  .

**Null hypothesis:** The sample mean is **equal** to the population mean.

**Alternative hypothesis**: The sample mean is **not equal** to the population mean.

## 2-Sample T-Tests *(The most important T-Test)*
- Comparing sample 1 to sample 2
- **“Is the mean of the x sample significantly different than the mean of the y sample.”**

>  .
> 
> $$ t = \frac{\bar{x_1} - \bar{x_2}}{\sqrt{s^2 \left( \frac{1}{n_1} + \frac{1}{n_2} \right)}}$$
>
> where $s^2$ is the pooled sample variance:
>
> $$ s^2 = \frac{\sum_{i=1}^{n_1} \left(x_i - \bar{x_1}\right)^2 + \sum_{j=1}^{n_2} \left(x_j - \bar{x_2}\right)^2 }{n_1 + n_2 - 2} $$
>  .


**Null hypothesis:** The two group means are **equal**.

**Alternative hypothesis:** The two group means are not **equal**.

```python
# Sample data for two groups
group1 = [25, 30, 35, 40, 45]
group2 = [20, 22, 24, 26, 28]

# Perform the two-sample t-test
t_stat, p_value = stats.ttest_ind(group1, group2)

# Print the results
print("t-statistic:", t_stat)
print("p-value:", p_value)  
```
```python
t-statistic: 3.3333333333333335
p-value: 0.014937765459710322
```

```python
alpha = 0.05  # Example significance level
if p_value < alpha:
    print("Reject the null hypothesis: There  
 is a significant difference between the means of the two groups.")
else:
    print("Fail to reject the null hypothesis: There is no significant difference between the means of the two groups.")  
```
```python
Reject the null hypothesis: There is a significant difference between the means of the two groups.
```


# ANOVA Test (F One-Way Test)

ANOVA works by comparing the variance of **group means** to the **variance within groups.** This process determines if the groups are part of one larger population or separate populations with different means.
- It tests continuous data, not discrete. 

- It allows us to test **3 or more groups at once**. 

F-tests compare the variance **between** groups to the variance **within** groups.

### Requirements:

The traditional F-test ANOVA assumes that **all groups have equal variances.**

**Null Hypothesis:** All of the group means are **equal**. 

**Alternative Hypothesis:** At least one mean is **different**. 

```python
# Sample data for three groups
group1 = [10, 15, 12, 18, 14] #mean: 13.8
group2 = [20, 25, 22, 28, 24] #mean: 22.8
group3 = [5, 8, 7, 10, 9] #mean: 7.8

# Perform the one-way ANOVA test
f_statistic, p_value = stats.f_oneway(group1, group2, group3)

# Print, round the values to 2 decimal places
f_statistic = round(f_statistic, 2)
p_value = round(p_value, 2)
```
```python
F-statistic: 44.34
p-value: 0.00 
```
In this case our p-value is incredibly small!

```python
alpha = 0.05  
if p_value < alpha:
    print("Reject the null hypothesis: There  
 is a significant difference between the means of the groups.")
else:
    print("Fail to reject the null hypothesis: There is no significant difference between the means of the groups.")  
```
```
Reject the null hypothesis: There is a significant difference between the means of the groups.
```
This confirms that yes, at least one mean is significantly different! However, it does not tell us which group (or groups) are the outliers. 
