# 5.2 Activity

## Random Number Generation, Inverse Transform Sampling & Monte Carlo

**Tools:** Python, NumPy, matplotlib, Jupyter/VS Code

## Learning objectives

By the end of this tutorial, you should be able to:

- use NumPy's modern random-number generator;
- reproduce simulations using a fixed seed;
- simulate from common distributions;
- verify simulation output against theory;
- estimate probabilities and expectations with Monte Carlo;
- calculate Monte Carlo standard errors;
- visualize convergence;
- vectorize repeated simulation;
- generate sampling distributions;
- implement inverse-transform sampling;
- simulate discrete variables from uniforms;
- evaluate estimators using repeated simulation;
- estimate bias, variance, MSE, and coverage.

---

# Part 0 — Set up

Create:

```text
notebooks/week10_simulation.ipynb
```

Import:

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
```

Create:

```python
rng = np.random.default_rng(556)
```

---

# Part 1 — Reproducible RNG

Run:

```python
rng = np.random.default_rng(556)

rng.normal(
    size=5
)
```

Restart:

```python
rng = np.random.default_rng(556)

rng.normal(
    size=5
)
```

### Question

Are the values identical?

Why?

---

# Part 2 — Different seeds

Compare:

```python
rng1 = np.random.default_rng(556)
rng2 = np.random.default_rng(557)
```

Generate:

```python
rng1.normal(size=5)
rng2.normal(size=5)
```

### Reflection

What role does the seed play?

---

# Part 3 — Simulate common distributions

Generate 10,000 draws from:

### Normal

```python
normal = rng.normal(
    10,
    2,
    size=10000
)
```

### Uniform

```python
uniform = rng.uniform(
    0,
    1,
    size=10000
)
```

### Binomial

```python
binomial = rng.binomial(
    n=10,
    p=0.4,
    size=10000
)
```

### Poisson

```python
poisson = rng.poisson(
    lam=3,
    size=10000
)
```

---

# Part 4 — Verify theoretical moments

For the normal sample:

```python
normal.mean()
normal.std()
```

For the uniform sample:

```python
uniform.mean()
uniform.var()
```

For the Poisson sample:

```python
poisson.mean()
poisson.var()
```

### Questions

1. How close are sample moments to theoretical values?
2. Why are they not exactly equal?

---

# Part 5 — Visualize distributions

Create histograms for the four simulated variables.

Example:

```python
plt.hist(
    normal,
    bins=30
)

plt.show()
```

### Reflection

Does the visual shape match the intended model?

---

# Part 6 — Randomness as a dependency

Write:

```python
def simulate_scores(
    n,
    mean,
    sd,
    rng
):
    return rng.normal(
        mean,
        sd,
        size=n
    )
```

Call:

```python
scores = simulate_scores(
    n=100,
    mean=80,
    sd=10,
    rng=rng
)
```

### Question

Why is passing `rng` preferable to creating a new seed inside the function?

---

# Part 7 — Monte Carlo probability

Estimate:

```text
P(Z > 1.96)
```

where:

```text
Z ~ Normal(0,1)
```

Use:

```python
z = rng.normal(
    size=100000
)

estimate = (
    z > 1.96
).mean()

estimate
```

Compare with the known probability, approximately `0.025`.

---

# Part 8 — Indicator interpretation

Inspect:

```python
(z > 1.96)[:10]
```

Then:

```python
(z > 1.96).astype(int)[:10]
```

Explain why the mean of the Boolean values estimates a probability.

---

# Part 9 — Estimate an expectation

Estimate:

```text
E[X²]
```

for:

```text
X ~ Uniform(0,1)
```

Use:

```python
x = rng.uniform(
    size=100000
)

estimate = np.mean(
    x ** 2
)
```

Compare with:

```text
1/3
```

---

# Part 10 — Monte Carlo standard error

Set:

```python
g = x ** 2
```

Compute:

```python
estimate = g.mean()

mcse = (
    g.std(ddof=1)
    / np.sqrt(len(g))
)

estimate
mcse
```

### Question

What uncertainty does `mcse` represent?

---

# Part 11 — Effect of simulation size

Repeat the same expectation estimate using:

```text
M = 100
M = 1,000
M = 10,000
M = 100,000
```

For each:

- estimate the expectation;
- calculate MCSE.

Store results in a DataFrame.

### Reflection

How does MCSE change as `M` increases?

---

# Part 12 — Running convergence

Generate:

```python
draws = rng.normal(
    size=10000
)
```

Compute:

```python
running_mean = (
    np.cumsum(draws)
    / np.arange(
        1,
        len(draws) + 1
    )
)
```

Plot:

```python
plt.plot(running_mean)
plt.axhline(0)
plt.xlabel("Number of draws")
plt.ylabel("Running mean")
plt.show()
```

### Question

What does the plot show?

---

# Part 13 — Loop vs. vectorized simulation

Loop:

```python
results = []

for _ in range(10000):
    value = rng.normal()
    results.append(
        value > 1.96
    )
```

Vectorized:

```python
values = rng.normal(
    size=10000
)

vectorized_result = (
    values > 1.96
).mean()
```

Compare the implementations.

---

# Part 14 — Benchmark

Use:

```python
%timeit
```

to compare loop and vectorized versions.

### Reflection

How does Week 9's vectorization material improve simulation workflows?

---

# Part 15 — Repeated samples

Generate:

```python
samples = rng.normal(
    loc=0,
    scale=1,
    size=(10000, 30)
)
```

Inspect:

```python
samples.shape
```

Calculate:

```python
means = samples.mean(
    axis=1
)
```

Inspect:

```python
means.shape
```

### Question

What does each row represent?

---

# Part 16 — Sampling distribution of the mean

Plot:

```python
plt.hist(
    means,
    bins=30
)

plt.show()
```

Check:

```python
means.mean()
means.std()
```

Compare with the theoretical standard error:

```text
1 / sqrt(30)
```

---

# Part 17 — CLT experiment

Generate exponential samples:

```python
samples = rng.exponential(
    scale=1,
    size=(10000, 30)
)

means = samples.mean(
    axis=1
)
```

Plot:

```python
plt.hist(
    means,
    bins=30
)

plt.show()
```

### Reflection

How does the sampling distribution compare with the source distribution?

---

# Part 18 — Inverse transform: exponential

Generate uniforms:

```python
u = rng.uniform(
    size=100000
)
```

Set:

```python
lam = 2
```

Transform:

```python
x_inverse = (
    -np.log(1 - u)
    / lam
)
```

Check:

```python
x_inverse.mean()
```

Compare with:

```text
1 / lam
```

---

# Part 19 — Compare with direct exponential sampling

Generate:

```python
x_direct = rng.exponential(
    scale=1 / lam,
    size=100000
)
```

Compare:

```python
x_inverse.mean()
x_direct.mean()
```

and:

```python
x_inverse.var()
x_direct.var()
```

Plot both distributions.

---

# Part 20 — Verify the inverse transform

For selected probabilities:

```python
probs = np.array([
    0.1,
    0.5,
    0.9
])
```

Compute:

```python
quantiles = (
    -np.log(1 - probs)
    / lam
)
```

Interpret the result as theoretical exponential quantiles.

---

# Part 21 — Bernoulli from uniforms

Generate:

```python
u = rng.uniform(
    size=10000
)
```

Create:

```python
bernoulli = (
    u < 0.3
).astype(int)
```

Check:

```python
bernoulli.mean()
```

Compare with:

```python
direct = rng.binomial(
    n=1,
    p=0.3,
    size=10000
)

direct.mean()
```

---

# Part 22 — Discrete inverse-transform sampling

Suppose:

```text
P(A)=0.2
P(B)=0.5
P(C)=0.3
```

Generate:

```python
u = rng.uniform(
    size=10000
)
```

Assign:

```python
outcome = np.where(
    u < 0.2,
    "A",
    np.where(
        u < 0.7,
        "B",
        "C"
    )
)
```

Check proportions:

```python
pd.Series(
    outcome
).value_counts(
    normalize=True
)
```

---

# Part 23 — Estimate π

Generate:

```python
xy = rng.uniform(
    -1,
    1,
    size=(100000, 2)
)
```

Determine:

```python
inside = (
    xy[:, 0] ** 2
    + xy[:, 1] ** 2
    <= 1
)
```

Estimate:

```python
pi_hat = (
    4 * inside.mean()
)

pi_hat
```

Compare with:

```python
np.pi
```

---

# Part 24 — Visualize the π experiment

Use a smaller sample:

```python
xy_small = rng.uniform(
    -1,
    1,
    size=(2000, 2)
)
```

Create an indicator for points inside the circle.

Plot the points with separate markers or categories for inside/outside.

### Question

What geometric probability is being estimated?

---

# Part 25 — Monte Carlo integration challenge

Estimate:

```text
∫₀¹ exp(-x²) dx
```

using:

```text
X ~ Uniform(0,1)
```

Your estimate is:

```python
x = rng.uniform(
    size=100000
)

estimate = np.mean(
    np.exp(
        -(x ** 2)
    )
)
```

Calculate an MCSE.

---

# Part 26 — Simulate an estimator

Set:

```python
MU = 5
SIGMA = 2
N = 20
N_SIM = 10000
```

Generate:

```python
samples = rng.normal(
    MU,
    SIGMA,
    size=(N_SIM, N)
)
```

Estimate:

```python
mu_hat = samples.mean(
    axis=1
)
```

Inspect:

```python
mu_hat.mean()
mu_hat.std()
```

---

# Part 27 — Bias

Calculate:

```python
bias = (
    mu_hat.mean()
    - MU
)

bias
```

### Question

Does the sample mean appear unbiased?

---

# Part 28 — Variance and MSE

Compute:

```python
variance = mu_hat.var(
    ddof=1
)
```

and:

```python
mse = np.mean(
    (
        mu_hat - MU
    ) ** 2
)
```

Compare:

```python
variance
mse
```

Why are they close when bias is close to zero?

---

# Part 29 — Compare two estimators

For each simulated sample calculate:

```python
mean_estimator = (
    samples.mean(axis=1)
)
```

and:

```python
median_estimator = (
    np.median(
        samples,
        axis=1
    )
)
```

Compare:

- bias;
- variance;
- MSE.

### Reflection

Which estimator performs better under normal data for estimating the population mean?

---

# Part 30 — Confidence interval coverage

For each sample compute:

```python
means = samples.mean(
    axis=1
)

ses = samples.std(
    axis=1,
    ddof=1
) / np.sqrt(N)
```

Create approximate 95% intervals:

```python
lower = means - 1.96 * ses
upper = means + 1.96 * ses
```

Check:

```python
covered = (
    (lower <= MU)
    & (MU <= upper)
)
```

Estimate:

```python
coverage = (
    covered.mean()
)

coverage
```

---

# Part 31 — Simulation function

Write:

```python
def simulate_mean_estimator(
    n,
    mu,
    sigma,
    n_sim,
    rng
):
    samples = rng.normal(
        mu,
        sigma,
        size=(n_sim, n)
    )

    return samples.mean(
        axis=1
    )
```

Test with:

```python
estimates = simulate_mean_estimator(
    n=20,
    mu=5,
    sigma=2,
    n_sim=10000,
    rng=rng
)
```

---

# Part 32 — Compare sample sizes

Run the function for:

```text
n = 5
n = 20
n = 100
```

For each calculate:

- bias;
- standard deviation;
- MSE.

Create a summary DataFrame.

### Question

How does estimator variability change with sample size?

---

# Part 33 — Avoid resetting seeds inside loops

Compare this bad pattern:

```python
results = []

for _ in range(5):
    rng_bad = np.random.default_rng(556)

    results.append(
        rng_bad.normal()
    )
```

Inspect:

```python
results
```

Now initialize once:

```python
rng_good = np.random.default_rng(556)

results = [
    rng_good.normal()
    for _ in range(5)
]
```

### Question

Why is the first pattern wrong?

---

# Part 34 — Simulation study design

Write a study plan before coding.

For example:

```text
Goal:
Data-generating distribution:
True parameter:
Sample size:
Number of simulations:
Estimator:
Performance measures:
Random seed:
```

Use your own design.

Then implement it.

---

# Part 35 — Mini simulation challenge

Investigate estimation of a population mean under a skewed distribution.

Use:

```text
X ~ Exponential(mean=1)
```

Compare:

```text
sample mean
sample median
```

for estimating:

```text
population mean = 1
```

Use:

```text
n = 10, 30, 100
```

and:

```text
10,000 simulations
```

For each estimator and sample size, report:

- bias;
- variance;
- MSE.

### Reflection

Does the median estimate the same population parameter as the mean?

This is an important conceptual check.

---

# Part 36 — Validate the simulation

For your mini study, verify:

```python
samples.shape
```

Check:

```python
np.isfinite(
    samples
).all()
```

Compare simulated moments with theoretical moments.

Check that probabilities are between:

```text
0 and 1
```

### Principle

> A simulation that runs is not necessarily a simulation of the intended model.

---

# Part 37 — Git checkpoint

Run:

```bash
git status
```

Stage:

```bash
git add .
```

Commit:

```bash
git commit -m "Complete Week 10 simulation and Monte Carlo exercises"
```

Push:

```bash
git push
```

---

# Part 38 — Final reflection

Answer in Markdown.

### 1. Randomness

What does a random seed control?

### 2. Generator design

Why pass an RNG object into a simulation function?

### 3. Monte Carlo

What is the central idea of Monte Carlo estimation?

### 4. Probability

Why does the mean of an indicator estimate a probability?

### 5. MCSE

What does Monte Carlo standard error measure?

### 6. Vectorization

Why is vectorization especially useful for simulation?

### 7. Sampling distributions

What does a repeated-sampling simulation approximate?

### 8. Inverse transform

How can Uniform(0,1) draws be transformed into another distribution?

### 9. Estimator evaluation

How do bias, variance, and MSE differ?

### 10. Reproducibility

Why is random-number management part of scientific reproducibility?

---

# Completion checklist

- [ ] Created Week 10 notebook
- [ ] Used `np.random.default_rng()`
- [ ] Reproduced results with a fixed seed
- [ ] Compared different seeds
- [ ] Simulated Normal, Uniform, Binomial, and Poisson variables
- [ ] Checked simulated moments against theory
- [ ] Visualized simulated distributions
- [ ] Passed an RNG object into a function
- [ ] Estimated a probability with Monte Carlo
- [ ] Interpreted Boolean indicators
- [ ] Estimated an expectation
- [ ] Calculated Monte Carlo standard error
- [ ] Compared simulation sizes
- [ ] Visualized running convergence
- [ ] Compared loop and vectorized simulation
- [ ] Simulated repeated samples in a 2D array
- [ ] Generated a sampling distribution
- [ ] Explored the CLT with skewed data
- [ ] Implemented exponential inverse-transform sampling
- [ ] Compared inverse-transform and direct sampling
- [ ] Simulated Bernoulli outcomes from uniforms
- [ ] Implemented a discrete inverse transform
- [ ] Estimated π by Monte Carlo
- [ ] Performed Monte Carlo integration
- [ ] Simulated an estimator
- [ ] Calculated bias
- [ ] Calculated variance
- [ ] Calculated MSE
- [ ] Compared two estimators
- [ ] Estimated confidence-interval coverage
- [ ] Wrote a reusable simulation function
- [ ] Compared estimator behavior across sample sizes
- [ ] Demonstrated why seeds should not be reset inside loops
- [ ] Designed a simulation study before coding
- [ ] Completed the mini simulation challenge
- [ ] Validated simulation output
- [ ] Committed Week 10 work to Git
- [ ] Pushed work to GitHub

---

# What you should now understand

```text
Probability model
      ↓
Pseudorandom generator
      ↓
Simulated samples
      ↓
Vectorized statistics
      ↓
Repeated experiments
      ↓
Monte Carlo estimate
      ↓
Simulation uncertainty
      ↓
Evaluate statistical procedures
```

Next week we will build **bootstrap and permutation procedures from scratch**, using the repeated-sampling framework developed here.
