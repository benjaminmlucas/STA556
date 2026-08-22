# 5.2 Simulation & Monte Carlo Methods

## Why this week matters

Week 9 focused on vectorized numerical computing with NumPy. Week 10 uses that machinery to study one of the most important tools in computational statistics:

> **simulation**

The approved STA 556 schedule identifies Week 10 as covering:

- random-number generation;
- inverse-transform sampling;
- Monte Carlo methods.

Simulation is fundamental because many statistical quantities are difficult or impossible to derive analytically, but can be approximated computationally.

Typical uses include:

- approximating probabilities;
- estimating expectations;
- evaluating estimators;
- exploring sampling distributions;
- approximating integrals;
- propagating uncertainty;
- validating statistical procedures.

---

# 1. What is simulation?

Simulation means generating artificial observations from a specified probabilistic model.

For example:

```python
import numpy as np

rng = np.random.default_rng(556)

x = rng.normal(
    loc=0,
    scale=1,
    size=1000
)
```

Here we simulate:

```text
X₁, ..., X₁₀₀₀ ~ Normal(0,1)
```

The computer generates values that behave like draws from the model.

---

# 2. Pseudorandom numbers

Computers do not usually generate truly random numbers algorithmically.

Instead, they generate **pseudorandom** sequences.

A pseudorandom generator:

```text
seed
  ↓
deterministic algorithm
  ↓
sequence that behaves statistically like random draws
```

This is valuable because the same seed can reproduce the same simulation.

---

# 3. Reproducible random-number generation

Prefer NumPy's generator API:

```python
rng = np.random.default_rng(556)
```

Then:

```python
rng.normal(size=5)
```

If the same seed is used again:

```python
rng = np.random.default_rng(556)
```

the same sequence is generated.

This makes simulation reproducible.

---

# 4. Why `default_rng()`?

Modern NumPy recommends generator objects rather than relying on global random state.

A generator makes randomness an explicit dependency.

Instead of:

```python
np.random.seed(556)
np.random.normal(size=10)
```

prefer:

```python
rng = np.random.default_rng(556)
rng.normal(size=10)
```

The generator can also be passed into functions.

---

# 5. Randomness as a function dependency

Consider:

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

Then:

```python
rng = np.random.default_rng(556)

scores = simulate_scores(
    n=100,
    mean=80,
    sd=10,
    rng=rng
)
```

The source of randomness is explicit.

This makes the function easier to test and reproduce.

---

# 6. Common distributions

NumPy generators support many useful distributions.

Normal:

```python
rng.normal(
    loc=0,
    scale=1,
    size=1000
)
```

Uniform:

```python
rng.uniform(
    low=0,
    high=1,
    size=1000
)
```

Binomial:

```python
rng.binomial(
    n=10,
    p=0.4,
    size=1000
)
```

Poisson:

```python
rng.poisson(
    lam=3,
    size=1000
)
```

Choice from categories:

```python
rng.choice(
    ["A", "B", "C"],
    size=100
)
```

---

# 7. Verify a simulation

If:

```python
x = rng.normal(
    10,
    2,
    size=100000
)
```

then check:

```python
x.mean()
x.std()
```

We expect approximately:

```text
mean ≈ 10
sd ≈ 2
```

Simulation output should be checked against theoretical expectations.

---

# 8. Simulation variability

If we simulate only:

```python
n = 10
```

the sample mean may differ considerably from the population mean.

With:

```python
n = 100000
```

the sample mean will usually be closer.

Simulation results themselves are random.

This leads to a central Monte Carlo idea:

> **Approximation error decreases as the number of simulations increases.**

---

# 9. Monte Carlo methods

A Monte Carlo method uses repeated random simulation to approximate a numerical or probabilistic quantity.

Generic workflow:

```text
Define target quantity
       ↓
Generate random samples
       ↓
Compute statistic
       ↓
Repeat many times
       ↓
Average / summarize
       ↓
Approximate target
```

---

# 10. Estimating a probability

Suppose:

```text
X ~ Normal(0,1)
```

and we want:

```text
P(X > 1.96)
```

Simulate:

```python
x = rng.normal(
    size=100000
)
```

Estimate:

```python
estimate = (
    x > 1.96
).mean()
```

Because Boolean values act like 0/1 indicators:

```text
True  → 1
False → 0
```

the mean estimates the probability.

---

# 11. Monte Carlo as indicator averaging

If:

```text
I(X > 1.96)
```

is an indicator, then:

```text
E[I(X > 1.96)]
=
P(X > 1.96)
```

Monte Carlo approximation:

```text
P(X > 1.96)
≈
(1/M) Σ I(X_m > 1.96)
```

This simple idea is extremely powerful.

---

# 12. Estimating expectations

Suppose:

```text
X ~ Uniform(0,1)
```

and we want:

```text
E[X²]
```

Simulate:

```python
x = rng.uniform(
    0,
    1,
    size=100000
)
```

Estimate:

```python
np.mean(
    x ** 2
)
```

Theoretical value:

```text
1/3
```

Monte Carlo approximates the expectation using a sample average.

---

# 13. Monte Carlo integration

Many integrals can be written as expectations.

For example:

```text
∫₀¹ x² dx
=
E[X²], where X ~ Uniform(0,1)
```

Therefore:

```python
x = rng.uniform(
    0,
    1,
    size=M
)

estimate = np.mean(
    x ** 2
)
```

approximates the integral.

---

# 14. Monte Carlo error

Suppose:

```text
θ̂_MC
```

is a Monte Carlo average from `M` simulated values.

Its standard error typically decreases at approximately:

```text
1 / √M
```

This means that reducing Monte Carlo error by a factor of 10 generally requires about 100 times as many simulations.

That is an important computational tradeoff.

---

# 15. Monte Carlo standard error

Suppose we simulate values:

```python
g = x ** 2
```

Then:

```python
estimate = g.mean()
```

A Monte Carlo standard error estimate is:

```python
mcse = (
    g.std(ddof=1)
    / np.sqrt(len(g))
)
```

This quantifies uncertainty from using a finite number of simulations.

---

# 16. Simulation size should be justified

Avoid choosing:

```text
10,000 simulations
```

simply because it sounds large.

Ask:

- What level of Monte Carlo precision is needed?
- How expensive is each simulation?
- Is the estimated quantity stable?
- What is the Monte Carlo standard error?

Simulation size is part of the analysis design.

---

# 17. Visualize convergence

Suppose:

```python
draws = rng.normal(
    size=10000
)
```

Compute cumulative means:

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
import matplotlib.pyplot as plt

plt.plot(running_mean)
plt.axhline(0)
plt.show()
```

The running estimate typically stabilizes as more draws accumulate.

---

# 18. Vectorized simulation

Less efficient:

```python
results = []

for _ in range(10000):
    x = rng.normal()
    results.append(x > 1.96)
```

Vectorized:

```python
x = rng.normal(
    size=10000
)

estimate = (
    x > 1.96
).mean()
```

Week 9's vectorization concepts become especially valuable in simulation.

---

# 19. Simulating repeated samples

Suppose we want 10,000 samples, each of size 30:

```python
samples = rng.normal(
    loc=0,
    scale=1,
    size=(10000, 30)
)
```

Then:

```python
sample_means = (
    samples.mean(axis=1)
)
```

This generates 10,000 sample means with one vectorized operation.

---

# 20. Sampling distributions

If:

```text
X₁, ..., Xₙ ~ distribution
```

then repeated samples generate a sampling distribution for a statistic.

Example:

```python
samples = rng.exponential(
    scale=1,
    size=(10000, 30)
)

means = samples.mean(
    axis=1
)
```

Even though the source distribution is skewed, the distribution of sample means may become approximately normal.

Simulation makes theoretical sampling behavior visible.

---

# 21. Inverse-transform sampling

Suppose we know a cumulative distribution function:

```text
F(x)
```

If:

```text
U ~ Uniform(0,1)
```

then under suitable conditions:

```text
X = F⁻¹(U)
```

has CDF `F`.

This is called **inverse-transform sampling**.

---

# 22. Why inverse-transform sampling works

Let:

```text
X = F⁻¹(U)
```

Then:

```text
P(X ≤ x)
=
P(F⁻¹(U) ≤ x)
```

which implies:

```text
P(U ≤ F(x))
=
F(x)
```

because:

```text
U ~ Uniform(0,1)
```

This turns uniform random numbers into samples from another distribution.

---

# 23. Exponential example

For:

```text
X ~ Exponential(rate = λ)
```

the CDF is:

```text
F(x) = 1 - exp(-λx)
```

Set:

```text
u = F(x)
```

and solve for `x`:

```text
x = -log(1-u) / λ
```

Therefore:

```python
u = rng.uniform(
    size=10000
)

lam = 2

x = (
    -np.log(1 - u)
    / lam
)
```

produces exponential draws.

---

# 24. Compare with NumPy

NumPy can sample directly:

```python
x_numpy = rng.exponential(
    scale=1 / lam,
    size=10000
)
```

Compare:

```python
x.mean()
x_numpy.mean()
```

Both should be close to:

```text
1 / λ
```

---

# 25. Inverse transform and numerical stability

Because:

```text
1 - U
```

is also Uniform(0,1), we can often write:

```python
x = (
    -np.log(u)
    / lam
)
```

provided:

```text
U ∈ (0,1)
```

Equivalent formulas may differ numerically at extreme values.

Numerical details matter even in conceptually simple algorithms.

---

# 26. Discrete inverse-transform sampling

Suppose a discrete variable takes:

```text
A with probability 0.2
B with probability 0.5
C with probability 0.3
```

Cumulative probabilities:

```text
A: 0.2
B: 0.7
C: 1.0
```

For:

```text
U ~ Uniform(0,1)
```

assign:

```text
U < 0.2       → A
0.2 ≤ U < 0.7 → B
0.7 ≤ U       → C
```

This is the discrete analogue of inverse-transform sampling.

---

# 27. Simulating Bernoulli trials

A Bernoulli variable can be generated using:

```python
u = rng.uniform(
    size=10000
)

x = (
    u < 0.3
).astype(int)
```

Then:

```python
x.mean()
```

should be near:

```text
0.3
```

NumPy's direct method:

```python
rng.binomial(
    n=1,
    p=0.3,
    size=10000
)
```

produces the same distribution.

---

# 28. Monte Carlo estimate of π

A classic example uses points sampled uniformly in a square.

Generate:

```python
xy = rng.uniform(
    -1,
    1,
    size=(100000, 2)
)
```

Inside the unit circle:

```python
inside = (
    xy[:, 0] ** 2
    + xy[:, 1] ** 2
    <= 1
)
```

The area ratio satisfies:

```text
π / 4
≈
proportion inside circle
```

Therefore:

```python
pi_hat = (
    4 * inside.mean()
)
```

---

# 29. Why the π example is useful

Estimating π by simulation is not computationally competitive with direct numerical methods.

Its value is conceptual.

It demonstrates:

- random sampling;
- geometric probability;
- indicator variables;
- Monte Carlo averaging;
- convergence;
- simulation error.

---

# 30. Evaluating an estimator by simulation

Suppose:

```text
X₁,...,Xₙ ~ Normal(μ, σ²)
```

and estimator:

```text
μ̂ = sample mean
```

We can simulate repeated datasets:

```python
samples = rng.normal(
    loc=5,
    scale=2,
    size=(10000, 20)
)
```

Compute:

```python
estimates = (
    samples.mean(axis=1)
)
```

Then examine:

```python
estimates.mean()
estimates.std()
```

Simulation lets us empirically study estimator properties.

---

# 31. Bias

For estimator:

```text
θ̂
```

bias is:

```text
Bias(θ̂)
=
E[θ̂] - θ
```

Simulation estimate:

```python
bias = (
    estimates.mean()
    - true_value
)
```

Repeated simulation can help assess whether an estimator is approximately unbiased.

---

# 32. Variance and MSE

Simulation can estimate:

```text
Var(θ̂)
```

using:

```python
estimates.var(
    ddof=1
)
```

Mean squared error:

```text
MSE
=
E[(θ̂ - θ)²]
```

estimated by:

```python
mse = np.mean(
    (
        estimates
        - true_value
    ) ** 2
)
```

---

# 33. Coverage probability

Suppose every simulated dataset produces a confidence interval:

```text
[L_m, U_m]
```

Coverage indicator:

```text
I(L_m ≤ θ ≤ U_m)
```

Estimated coverage:

```python
coverage = (
    covered.mean()
)
```

This lets us evaluate whether a nominal:

```text
95%
```

confidence interval actually covers the true parameter about 95% of the time under the simulation model.

---

# 34. Simulation studies are experiments

A good simulation study specifies:

```text
data-generating process
parameter values
sample size
number of replications
estimator/procedure
performance metric
random-number strategy
```

Treat the simulation design as seriously as an empirical experiment.

---

# 35. Separate simulation design from execution

Instead of hard-coding:

```python
samples = rng.normal(
    5,
    2,
    size=(10000, 20)
)
```

define:

```python
MU = 5
SIGMA = 2
N = 20
N_SIM = 10000
```

Then:

```python
samples = rng.normal(
    MU,
    SIGMA,
    size=(N_SIM, N)
)
```

Named parameters make simulation studies easier to audit.

---

# 36. Simulation functions

A reusable simulation function might be:

```python
def simulate_mean(
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

Then:

```python
estimates = simulate_mean(
    n=20,
    mu=5,
    sigma=2,
    n_sim=10000,
    rng=rng
)
```

---

# 37. Independent random streams

Large simulation projects may need separate random streams.

A simple practical rule is:

> Do not repeatedly reset the seed inside a loop.

Bad:

```python
for i in range(100):
    rng = np.random.default_rng(556)
    ...
```

This regenerates the same sequence repeatedly.

Instead, initialize once and draw sequentially.

---

# 38. Parallel simulation requires care

If simulations are parallelized, each worker needs an appropriate independent random stream.

Naively giving every worker the same seed can produce duplicated simulations.

This becomes important in later performance-oriented work.

For Week 10, the main principle is:

> **Random-number management is part of reproducible software design.**

---

# 39. Validate a simulation

Always ask:

- Do simulated means match theory?
- Are probabilities in `[0,1]`?
- Is the output shape correct?
- Are values in the correct support?
- Does a histogram resemble the intended distribution?
- Does increasing simulation size stabilize the estimate?

Simulation code can run successfully and still simulate the wrong model.

---

# 40. Key ideas

By the end of Week 10, you should be able to explain:

1. What simulation means in statistical computing.
2. What pseudorandom numbers are.
3. Why random seeds support reproducibility.
4. Why NumPy generator objects are useful.
5. How to generate common distributions.
6. What Monte Carlo estimation is.
7. How probabilities can be estimated using indicators.
8. How expectations can be estimated using sample averages.
9. How Monte Carlo integration works.
10. Why Monte Carlo error decreases roughly as `1/sqrt(M)`.
11. How to estimate Monte Carlo standard error.
12. Why vectorization is useful for simulation.
13. What a sampling distribution is.
14. How inverse-transform sampling works.
15. How to derive exponential inverse-transform sampling.
16. How to simulate discrete outcomes from uniforms.
17. How to evaluate estimators by simulation.
18. Bias, variance, MSE, and coverage in simulation studies.
19. Why simulation design parameters should be explicit.
20. Why random-number management matters in larger computational workflows.

---

# 41. Recommended reading

## NumPy Random Sampling

Official NumPy documentation for modern random-number generation.

https://numpy.org/doc/stable/reference/random/index.html

## NumPy Generator

Reference for the `Generator` API and supported distributions.

https://numpy.org/doc/stable/reference/random/generator.html

## SciPy Statistical Distributions

Useful reference for probability distributions, CDFs, inverse CDFs, and random variate generation.

https://docs.scipy.org/doc/scipy/reference/stats.html

## Python Data Science Handbook — NumPy

Useful background on vectorized numerical operations, which directly support efficient Monte Carlo computation.

https://jakevdp.github.io/PythonDataScienceHandbook/

---

# 42. YouTube recommendations

## 1. jbstatistics — "Monte Carlo Simulation"

A clear conceptual introduction to Monte Carlo simulation and repeated random sampling. It is useful for reinforcing the idea that simulation approximates probabilities and expectations through repeated experiments.

**Recommended use:** Watch before the Monte Carlo estimation section.

[Find on YouTube](https://www.youtube.com/results?search_query=jbstatistics+Monte+Carlo+Simulation)

---

## 2. Inverse Transform Sampling

A focused tutorial on inverse-transform sampling is particularly useful this week because the method connects uniform random-number generation to arbitrary target distributions.

**Recommended use:** Watch before the inverse-transform sampling exercises.

[Find inverse-transform sampling tutorials on YouTube](https://www.youtube.com/results?search_query=inverse+transform+sampling+tutorial)

---

## 3. NumPy Random Number Generation

A practical NumPy tutorial on `default_rng`, distributions, seeds, and reproducible random-number generation is useful for the implementation side of the week.

**Recommended use:** Optional reinforcement before starting the hands-on activity.

[Find NumPy RNG tutorials on YouTube](https://www.youtube.com/results?search_query=NumPy+default_rng+random+number+generation+tutorial)

---

# 43. Week 10 takeaway

The central lesson is:

> **Simulation converts probability models into computational experiments.**

The progression is:

```text
Probability model
      ↓
Random-number generator
      ↓
Simulated samples
      ↓
Statistic / indicator
      ↓
Repeat
      ↓
Monte Carlo approximation
      ↓
Quantify simulation error
      ↓
Statistical insight
```

Next week we will use repeated simulation ideas to build **bootstrap and permutation procedures from scratch**.
