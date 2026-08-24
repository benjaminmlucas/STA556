# 5.3 Activity

## Bootstrap & Permutation Procedures from Scratch

**Tools:** Python, NumPy, pandas, matplotlib, SciPy, Jupyter/VS Code

## Course computing environment: GitHub Codespaces

For STA 556, the **officially supported computing environment is the course GitHub Codespace**. This gives everyone the same Linux/Python environment whether you are using a Mac, a Windows PC, or a university computer.

Before beginning this activity:

1. Open the repository for your course/assignment on GitHub.
2. Open or create its Codespace.
3. In VS Code, make sure the **repository root** is the folder open in the Explorer.
4. Open an integrated terminal and run:

```bash
pwd
git status
```

Your working directory should be the repository under `/workspaces/...`, and `git status` should recognize the repository.

Unless an activity explicitly says otherwise:

- run terminal commands from the **repository root**;
- use repository-relative paths such as `data/...`, `src/...`, and `notebooks/...`;
- use `python` for Python commands;
- do **not** run `git init` inside the course repository;
- do **not** install packages manually just because an import fails—first check the course environment and `requirements.txt`;
- commit and push meaningful work regularly.

If this is your first time using Codespaces, read `0_Codespaces_Introduction.md` before continuing.

---

## Learning objectives

By the end of this tutorial, you should be able to:

-   generate bootstrap samples manually;
-   construct bootstrap distributions;
-   estimate bootstrap standard errors;
-   construct percentile bootstrap confidence intervals;
-   bootstrap means, medians, and paired statistics;
-   preserve observational units during resampling;
-   create permutation null distributions;
-   calculate one- and two-sided permutation p-values;
-   distinguish exact and Monte Carlo permutation logic;
-   work with paired resampling structures;
-   write reusable resampling functions;
-   compare from-scratch results with SciPy.

------------------------------------------------------------------------

# Part 0 --- Set up

Create:

``` text
notebooks/week11_resampling.ipynb
```

Import:

``` python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
```

Create:

``` python
rng = np.random.default_rng(556)
```

Define:

``` python
N_BOOT = 10000
N_PERM = 10000
```

------------------------------------------------------------------------

# Part 1 --- A small observed sample

Create:

``` python
x = np.array([
    4.2,
    5.1,
    3.8,
    6.0,
    4.9,
    5.4,
    4.5,
    5.7
])
```

Inspect:

``` python
x.mean()
np.median(x)
x.std(ddof=1)
```

### Question

What do we know about the population that generated `x`?

What do we *not* know?

------------------------------------------------------------------------

# Part 2 --- One bootstrap sample

Run:

``` python
bootstrap_sample = rng.choice(
    x,
    size=len(x),
    replace=True
)
```

Inspect:

``` python
bootstrap_sample
```

### Questions

1.  Is the sample the same size as `x`?
2.  Do any observations repeat?
3.  Are any original observations absent?

------------------------------------------------------------------------

# Part 3 --- Why replacement matters

Try:

``` python
rng.choice(
    x,
    size=len(x),
    replace=False
)
```

### Question

What has this actually done?

Why does it fail to create meaningful bootstrap variation in the sample
values?

------------------------------------------------------------------------

# Part 4 --- One bootstrap statistic

Calculate:

``` python
bootstrap_mean = (
    bootstrap_sample.mean()
)
```

Compare:

``` python
x.mean()
bootstrap_mean
```

The bootstrap mean is one replicate of the estimator.

------------------------------------------------------------------------

# Part 5 --- Bootstrap distribution with a loop

Create:

``` python
bootstrap_means = np.empty(
    N_BOOT
)
```

Then:

``` python
for b in range(N_BOOT):
    sample_b = rng.choice(
        x,
        size=len(x),
        replace=True
    )

    bootstrap_means[b] = (
        sample_b.mean()
    )
```

Inspect:

``` python
bootstrap_means[:10]
```

------------------------------------------------------------------------

# Part 6 --- Visualize the bootstrap distribution

Plot:

``` python
plt.hist(
    bootstrap_means,
    bins=30
)

plt.axvline(
    x.mean()
)

plt.xlabel("Bootstrap mean")
plt.ylabel("Frequency")
plt.show()
```

### Reflection

How does the bootstrap distribution differ from a histogram of the
original observations?

------------------------------------------------------------------------

# Part 7 --- Bootstrap standard error

Calculate:

``` python
bootstrap_se = (
    bootstrap_means.std(
        ddof=1
    )
)
```

Now calculate the ordinary analytical SE:

``` python
analytical_se = (
    x.std(ddof=1)
    / np.sqrt(len(x))
)
```

Compare:

``` python
bootstrap_se
analytical_se
```

Why are they similar but not identical?

------------------------------------------------------------------------

# Part 8 --- Percentile confidence interval

Compute:

``` python
bootstrap_ci = np.quantile(
    bootstrap_means,
    [0.025, 0.975]
)

bootstrap_ci
```

Add these limits to the histogram.

### Question

What does the percentile procedure actually do computationally?

------------------------------------------------------------------------

# Part 9 --- Bootstrap bias

Calculate:

``` python
bootstrap_bias = (
    bootstrap_means.mean()
    - x.mean()
)

bootstrap_bias
```

Interpret the sign.

------------------------------------------------------------------------

# Part 10 --- Vectorized bootstrap

Generate:

``` python
bootstrap_samples = rng.choice(
    x,
    size=(
        N_BOOT,
        len(x)
    ),
    replace=True
)
```

Inspect:

``` python
bootstrap_samples.shape
```

Calculate:

``` python
bootstrap_means_vec = (
    bootstrap_samples.mean(
        axis=1
    )
)
```

Compare:

``` python
bootstrap_means_vec.mean()
bootstrap_means_vec.std(ddof=1)
```

with the loop implementation.

------------------------------------------------------------------------

# Part 11 --- Memory discussion

Estimate how many values would be created by:

``` text
1,000,000 bootstrap samples
×
10,000 observations
```

### Reflection

Why might a loop or batched method be preferable even if vectorization
is faster?

------------------------------------------------------------------------

# Part 12 --- Bootstrap the median

Construct:

``` python
bootstrap_medians = np.empty(
    N_BOOT
)
```

Fill it using repeated bootstrap samples.

Then calculate:

``` python
median_se = (
    bootstrap_medians.std(
        ddof=1
    )
)

median_ci = np.quantile(
    bootstrap_medians,
    [0.025, 0.975]
)
```

### Question

Why is bootstrapping particularly attractive for statistics without a
simple elementary SE formula?

------------------------------------------------------------------------

# Part 13 --- Write a generic bootstrap function

Write:

``` python
def bootstrap_statistic(
    data,
    statistic,
    n_boot,
    rng
):
    results = np.empty(
        n_boot
    )

    for b in range(n_boot):
        sample = rng.choice(
            data,
            size=len(data),
            replace=True
        )

        results[b] = statistic(
            sample
        )

    return results
```

Test:

``` python
means = bootstrap_statistic(
    x,
    np.mean,
    N_BOOT,
    rng
)
```

Then:

``` python
medians = bootstrap_statistic(
    x,
    np.median,
    N_BOOT,
    rng
)
```

### Connection

Which Week 7 concept allows `statistic` itself to be passed as an
argument?

------------------------------------------------------------------------

# Part 14 --- Bootstrap rows, not columns

Create:

``` python
df = pd.DataFrame({
    "age": [22, 25, 31, 35, 41, 48],
    "score": [70, 75, 80, 86, 91, 94]
})
```

Suppose we want the correlation.

Observed:

``` python
observed_r = (
    df["age"]
    .corr(
        df["score"]
    )
)
```

### Wrong idea

Resampling the two columns separately would destroy the original
age-score pairs.

------------------------------------------------------------------------

# Part 15 --- Bootstrap paired rows

Generate bootstrap indices:

``` python
indices = rng.choice(
    len(df),
    size=len(df),
    replace=True
)
```

Use:

``` python
sample_b = df.iloc[
    indices
]
```

Compute:

``` python
sample_b[
    "age"
].corr(
    sample_b["score"]
)
```

### Question

Why do row indices preserve the data structure?

------------------------------------------------------------------------

# Part 16 --- Bootstrap a correlation

Repeat row-resampling `N_BOOT` times.

Store:

``` python
bootstrap_correlations
```

Then calculate:

-   bootstrap SE;
-   percentile 95% CI.

Plot the distribution.

------------------------------------------------------------------------

# Part 17 --- Two-group data

Create:

``` python
group_a = np.array([
    82, 88, 91, 85,
    90, 87, 93, 89
])

group_b = np.array([
    75, 80, 78, 84,
    79, 77, 81, 76
])
```

Calculate:

``` python
observed_difference = (
    group_a.mean()
    - group_b.mean()
)

observed_difference
```

------------------------------------------------------------------------

# Part 18 --- State the null hypothesis

Write in words:

``` text
H0:
```

For this simple permutation test, the null mechanism treats the
observations as exchangeable between groups.

### Question

What would the group labels mean if `H0` were true?

------------------------------------------------------------------------

# Part 19 --- Pool the observations

Create:

``` python
combined = np.concatenate([
    group_a,
    group_b
])
```

Inspect:

``` python
combined
len(combined)
```

Record:

``` python
n_a = len(group_a)
```

------------------------------------------------------------------------

# Part 20 --- One permutation

Run:

``` python
permuted = rng.permutation(
    combined
)
```

Split:

``` python
a_perm = permuted[
    :n_a
]

b_perm = permuted[
    n_a:
]
```

Calculate:

``` python
perm_difference = (
    a_perm.mean()
    - b_perm.mean()
)
```

Compare with the observed difference.

------------------------------------------------------------------------

# Part 21 --- Build the permutation distribution

Create:

``` python
null_distribution = np.empty(
    N_PERM
)
```

Then:

``` python
for b in range(N_PERM):
    permuted = rng.permutation(
        combined
    )

    a_perm = permuted[
        :n_a
    ]

    b_perm = permuted[
        n_a:
    ]

    null_distribution[b] = (
        a_perm.mean()
        - b_perm.mean()
    )
```

------------------------------------------------------------------------

# Part 22 --- Visualize the null distribution

Plot:

``` python
plt.hist(
    null_distribution,
    bins=30
)

plt.axvline(
    observed_difference
)

plt.xlabel(
    "Difference in means under H0"
)

plt.ylabel("Frequency")
plt.show()
```

### Question

Where is the null distribution centered?

Why?

------------------------------------------------------------------------

# Part 23 --- One-sided p-value

Suppose:

``` text
H1: mean_A > mean_B
```

Calculate:

``` python
p_greater = (
    1
    + np.sum(
        null_distribution
        >= observed_difference
    )
) / (
    N_PERM + 1
)

p_greater
```

Interpret.

------------------------------------------------------------------------

# Part 24 --- Two-sided p-value

Calculate:

``` python
p_two_sided = (
    1
    + np.sum(
        np.abs(
            null_distribution
        )
        >= np.abs(
            observed_difference
        )
    )
) / (
    N_PERM + 1
)

p_two_sided
```

### Question

What does "at least as extreme" mean here?

------------------------------------------------------------------------

# Part 25 --- Why add one?

Compare:

``` python
np.mean(
    null_distribution
    >= observed_difference
)
```

with the corrected Monte Carlo expression.

### Reflection

Why is reporting a p-value of exactly zero inappropriate when only a
finite random sample of permutations was generated?

------------------------------------------------------------------------

# Part 26 --- Change the number of permutations

Repeat the p-value estimation using:

``` text
100
1,000
10,000
```

Store each result.

### Question

How stable is the estimated p-value?

When might more permutations matter?

------------------------------------------------------------------------

# Part 27 --- Write a statistic function

Create:

``` python
def difference_in_means(
    x,
    y
):
    return (
        np.mean(x)
        - np.mean(y)
    )
```

Use it for both:

``` python
observed_difference
```

and every permutation replicate.

### Principle

Separate:

``` text
statistic
```

from:

``` text
resampling mechanism
```

------------------------------------------------------------------------

# Part 28 --- Generic independent permutation function

Write:

``` python
def permutation_distribution(
    x,
    y,
    statistic,
    n_perm,
    rng
):
    combined = np.concatenate([
        x,
        y
    ])

    n_x = len(x)

    results = np.empty(
        n_perm
    )

    for b in range(n_perm):
        permuted = rng.permutation(
            combined
        )

        x_perm = permuted[
            :n_x
        ]

        y_perm = permuted[
            n_x:
        ]

        results[b] = statistic(
            x_perm,
            y_perm
        )

    return results
```

Test it.

------------------------------------------------------------------------

# Part 29 --- Compare statistics

Repeat the permutation procedure using:

``` python
def difference_in_medians(
    x,
    y
):
    return (
        np.median(x)
        - np.median(y)
    )
```

Compare the resulting null distribution and p-value with the difference
in means.

### Reflection

The permutation mechanism is unchanged.

What changed?

------------------------------------------------------------------------

# Part 30 --- Small exact permutation example

Use:

``` python
x_small = np.array([
    1, 2
])

y_small = np.array([
    4, 5
])
```

Pool:

``` text
1, 2, 4, 5
```

Manually list all distinct ways to choose two observations for the first
group.

How many are there?

Hint:

``` text
C(4,2)
```

Calculate the difference in means for every assignment.

This is an exact permutation distribution.

------------------------------------------------------------------------

# Part 31 --- Exact vs. Monte Carlo

Compare:

``` text
all possible assignments
```

with:

``` text
randomly sampled assignments
```

### Questions

1.  When is exact enumeration feasible?
2.  Why does it rapidly become impossible as sample size increases?
3.  What approximation is introduced by Monte Carlo permutation?

------------------------------------------------------------------------

# Part 32 --- Paired data

Create:

``` python
before = np.array([
    80, 72, 88, 91, 77, 85
])

after = np.array([
    84, 75, 90, 94, 76, 89
])
```

Calculate:

``` python
differences = (
    after - before
)
```

Inspect:

``` python
differences
differences.mean()
```

------------------------------------------------------------------------

# Part 33 --- Preserve the pairing

### Incorrect

Pool all before/after observations and randomly split them.

### Better for a paired sign-randomization logic

For each difference, randomly choose:

``` text
+d_i
or
-d_i
```

Generate signs:

``` python
signs = rng.choice(
    [-1, 1],
    size=len(differences)
)

permuted_difference = (
    differences
    * signs
)
```

Calculate its mean.

------------------------------------------------------------------------

# Part 34 --- Paired null distribution

Repeat the random sign flip `N_PERM` times.

Store mean differences.

Compare the observed paired mean difference with the resulting null
distribution.

### Question

Why does this preserve more of the original study design than pooling
all observations?

------------------------------------------------------------------------

# Part 35 --- Validate with SciPy: permutation

Import:

``` python
from scipy.stats import (
    permutation_test
)
```

Define a compatible statistic:

``` python
def statistic(
    x,
    y
):
    return (
        np.mean(x)
        - np.mean(y)
    )
```

Run:

``` python
result = permutation_test(
    (
        group_a,
        group_b
    ),
    statistic,
    n_resamples=N_PERM,
    alternative="two-sided",
    rng=np.random.default_rng(
        556
    )
)
```

Inspect:

``` python
result.statistic
result.pvalue
```

Compare with your own implementation.

------------------------------------------------------------------------

# Part 36 --- Validate with SciPy: bootstrap

Import:

``` python
from scipy.stats import bootstrap
```

Run an appropriate bootstrap for the sample mean.

Compare its confidence interval with your percentile interval.

### Reflection

Do not expect every library default to match your from-scratch choices
exactly.

Check:

-   interval method;
-   confidence level;
-   RNG;
-   number of resamples.

------------------------------------------------------------------------

# Part 37 --- Bootstrap vs. permutation challenge

For each goal, choose **bootstrap** or **permutation**.

### A

Estimate the standard error of the median.

### B

Test whether two randomized groups differ in mean outcome under a
no-group-effect null.

### C

Estimate uncertainty in a regression coefficient.

### D

Construct a null distribution for a difference in groups.

### E

Estimate a confidence interval for a correlation.

Explain each choice.

------------------------------------------------------------------------

# Part 38 --- Mini resampling study

Create:

``` python
rng = np.random.default_rng(556)

control = rng.normal(
    70,
    10,
    size=30
)

treatment = rng.normal(
    76,
    10,
    size=30
)
```

Complete:

## Bootstrap

1.  Estimate the observed difference in means.
2.  Bootstrap the two groups appropriately.
3.  Construct the bootstrap distribution of the difference.
4.  Estimate the bootstrap SE.
5.  Construct a percentile 95% CI.

## Permutation

6.  Pool the observations.
7.  Construct a permutation null distribution.
8.  Estimate a two-sided p-value.

## Interpretation

9.  Explain why the two empirical distributions answer different
    questions even though both contain differences in means.

------------------------------------------------------------------------

# Part 39 --- Reproducibility and Monte Carlo stability

Repeat the mini study using:

``` text
1,000 resamples
10,000 resamples
```

Compare:

-   SE;
-   CI endpoints;
-   p-value.

### Question

Which quantities are most sensitive to the number of resamples?

------------------------------------------------------------------------

# Part 40 --- Resampling audit

Before trusting a procedure, answer:

``` text
What is the observational unit?
What statistic is being computed?
What is resampled?
Is replacement used?
What null assumption justifies permutation?
Is pairing/clustering preserved?
How many resamples are used?
How is randomness controlled?
What empirical distribution is being constructed?
What inferential quantity is extracted?
```

------------------------------------------------------------------------

# Part 41 --- Git checkpoint

Run:

``` bash
git status
```

Stage:

``` bash
git add .
```

Commit:

``` bash
git commit -m "Complete Week 11 bootstrap and permutation exercises"
```

Push:

``` bash
git push
```

------------------------------------------------------------------------

# Part 42 --- Final reflection

Answer in Markdown.

### 1. Bootstrap

Why does a bootstrap sample use replacement?

### 2. Empirical population

What population does the nonparametric bootstrap sample from?

### 3. Bootstrap distribution

What does it approximate?

### 4. Standard error

How do you estimate a bootstrap SE?

### 5. Percentile interval

How is it constructed?

### 6. Permutation

What does a permutation null distribution represent?

### 7. Exchangeability

Why is it central to a permutation test?

### 8. p-value

How is a Monte Carlo permutation p-value calculated?

### 9. Pairing

Why can we not arbitrarily shuffle observations in a paired study?

### 10. Comparison

What is the most important conceptual difference between bootstrap and
permutation resampling?

------------------------------------------------------------------------

# Completion checklist

-   [ ] Created Week 11 notebook
-   [ ] Generated one bootstrap sample
-   [ ] Explained why replacement is required
-   [ ] Constructed a bootstrap distribution with a loop
-   [ ] Visualized the bootstrap distribution
-   [ ] Calculated bootstrap SE
-   [ ] Compared bootstrap and analytical SE
-   [ ] Constructed a percentile CI
-   [ ] Estimated bootstrap bias
-   [ ] Implemented vectorized bootstrap sampling
-   [ ] Discussed memory tradeoffs
-   [ ] Bootstrapped a median
-   [ ] Wrote a generic bootstrap function
-   [ ] Preserved row pairings when resampling
-   [ ] Bootstrapped a correlation
-   [ ] Calculated an observed group difference
-   [ ] Constructed one random permutation
-   [ ] Built a permutation null distribution
-   [ ] Calculated one-sided p-value
-   [ ] Calculated two-sided p-value
-   [ ] Used a finite-resampling correction
-   [ ] Examined permutation-count stability
-   [ ] Wrote reusable statistic functions
-   [ ] Wrote a generic permutation function
-   [ ] Compared mean and median test statistics
-   [ ] Constructed a small exact permutation distribution
-   [ ] Compared exact and Monte Carlo permutation tests
-   [ ] Preserved paired design using sign randomization
-   [ ] Validated permutation output with SciPy
-   [ ] Validated bootstrap output with SciPy
-   [ ] Completed the bootstrap-vs-permutation challenge
-   [ ] Completed the mini resampling study
-   [ ] Compared results across resampling counts
-   [ ] Audited the resampling design
-   [ ] Committed Week 11 work to Git
-   [ ] Pushed work to GitHub

------------------------------------------------------------------------

# What you should now understand

``` text
Observed data
      ↓
Choose statistic
      ↓
Bootstrap
   sample with replacement
      ↓
sampling uncertainty
      ↓
SE / confidence interval

or

Permutation
   rearrange under H0
      ↓
null distribution
      ↓
p-value
```

Next week we move into **numerical optimization**, including root
finding, Newton--Raphson, gradient descent, and statistical objective
functions.
