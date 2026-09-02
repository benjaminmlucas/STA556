# 5.3 Bootstrap & Permutation Methods

**Course:** STA 556 Statistics and Data Science Computing Workflows\
**Program:** M.S. Statistics and Data Science, Northern Arizona
University\
**Week 11:** Statistical Computing --- Resampling Methods from Scratch

## Why this week matters

Week 10 introduced repeated simulation from a known probability model.
Week 11 changes the source of randomness.

Instead of saying:

``` text
Assume X ~ Normal(...)
and simulate from that model
```

we use the **observed data themselves** to generate reference
distributions.

The central question is:

> **How can we approximate uncertainty and null distributions by
> repeatedly resampling the data we actually observed?**

This week focuses on:

-   the resampling idea;
-   bootstrap resampling;
-   bootstrap distributions;
-   bootstrap standard errors;
-   bootstrap confidence intervals;
-   permutation tests;
-   permutation null distributions;
-   Monte Carlo p-values;
-   implementing both methods from scratch.

------------------------------------------------------------------------

# 1. The common resampling idea

Both bootstrap and permutation methods repeatedly recompute a statistic.

Generic structure:

``` text
Observed data
     ↓
Resample / rearrange
     ↓
Calculate statistic
     ↓
Repeat many times
     ↓
Empirical reference distribution
```

The key difference is **what operation generates the resampled data**.

------------------------------------------------------------------------

# 2. Bootstrap vs. permutation

## Bootstrap

Sample observations **with replacement** from the observed sample.

Goal:

``` text
approximate the sampling distribution
of an estimator/statistic
```

## Permutation

Rearrange labels or observations under a null hypothesis.

Goal:

``` text
approximate the distribution of a test statistic
when the null hypothesis is true
```

A useful summary:

``` text
bootstrap   → uncertainty
permutation → hypothesis testing
```

This is an oversimplification, but a useful starting point.

------------------------------------------------------------------------

# 3. Sampling with replacement

Suppose:

``` python
x = np.array([
    4.2,
    5.1,
    3.8,
    6.0,
    4.9
])
```

A bootstrap sample has the same size:

``` text
n = 5
```

but observations are drawn **with replacement**.

One bootstrap sample might be:

``` text
[5.1, 5.1, 3.8, 6.0, 5.1]
```

Some original observations may appear multiple times.

Others may not appear at all.

------------------------------------------------------------------------

# 4. Why with replacement?

The observed sample acts as an empirical approximation to the unknown
population distribution.

Conceptually:

``` text
Unknown population
      ↓
observed sample
      ↓
empirical distribution
      ↓
bootstrap samples
```

Sampling with replacement means each observed value can be selected
repeatedly, just as repeated independent observations from a population
could take similar values.

------------------------------------------------------------------------

# 5. One bootstrap replicate

Suppose the statistic is the sample mean.

``` python
rng = np.random.default_rng(556)

bootstrap_sample = rng.choice(
    x,
    size=len(x),
    replace=True
)

bootstrap_mean = (
    bootstrap_sample.mean()
)
```

That produces **one bootstrap replicate**.

------------------------------------------------------------------------

# 6. The bootstrap distribution

Repeat the procedure many times:

``` python
B = 10000

bootstrap_means = np.empty(B)

for b in range(B):
    sample_b = rng.choice(
        x,
        size=len(x),
        replace=True
    )

    bootstrap_means[b] = (
        sample_b.mean()
    )
```

The resulting values form the:

``` text
bootstrap distribution
```

of the sample mean.

------------------------------------------------------------------------

# 7. What does the bootstrap distribution approximate?

The true sampling distribution asks:

> If we repeatedly sampled new datasets from the population, how would
> the estimator vary?

But we usually only have one observed sample.

The bootstrap approximates this repeated-sampling process by repeatedly
sampling from the **empirical distribution** of the observed data.

------------------------------------------------------------------------

# 8. Bootstrap standard error

The bootstrap estimate of the standard error is simply the standard
deviation of the bootstrap replicates:

``` python
bootstrap_se = (
    bootstrap_means.std(
        ddof=1
    )
)
```

Conceptually:

``` text
bootstrap SE
≈
SD of estimator across repeated samples
```

------------------------------------------------------------------------

# 9. Compare with an analytical SE

For the sample mean:

``` text
SE(x̄) = s / √n
```

Compute:

``` python
analytical_se = (
    x.std(ddof=1)
    / np.sqrt(len(x))
)
```

Compare with:

``` python
bootstrap_se
```

For well-behaved problems, these may be similar.

The bootstrap becomes especially valuable when an analytical
standard-error formula is difficult or unavailable.

------------------------------------------------------------------------

# 10. Bootstrap confidence intervals

A simple approach is the **percentile bootstrap interval**.

For a 95% interval:

``` python
lower, upper = np.quantile(
    bootstrap_means,
    [0.025, 0.975]
)
```

Interpretation:

``` text
take the middle 95% of the bootstrap distribution
```

This is straightforward, but it is not the only bootstrap
confidence-interval method.

------------------------------------------------------------------------

# 11. Bootstrap confidence intervals are not all equivalent

Common bootstrap intervals include:

``` text
percentile
basic
studentized
BCa
```

This week focuses primarily on the **percentile interval** because
students are implementing the procedure from scratch.

The important lesson is:

> Bootstrap resampling is a general computational framework; the
> inferential method built from it still requires statistical choices.

------------------------------------------------------------------------

# 12. Bootstrap bias

Suppose:

``` python
observed_stat = x.mean()
```

and:

``` python
bootstrap_mean_stat = (
    bootstrap_means.mean()
)
```

Estimate bootstrap bias:

``` python
bootstrap_bias = (
    bootstrap_mean_stat
    - observed_stat
)
```

This approximates:

``` text
E(θ̂*) - θ̂
```

where `θ̂*` denotes a bootstrap replicate.

------------------------------------------------------------------------

# 13. Bootstrap any statistic

The bootstrap is not limited to means.

Examples:

``` text
median
correlation
regression coefficient
difference in means
quantiles
classification performance
```

As long as the statistic can be computed on a resampled dataset, a
bootstrap distribution can often be constructed.

------------------------------------------------------------------------

# 14. Bootstrap the median

Example:

``` python
bootstrap_medians = np.empty(B)

for b in range(B):
    sample_b = rng.choice(
        x,
        size=len(x),
        replace=True
    )

    bootstrap_medians[b] = (
        np.median(sample_b)
    )
```

This is useful because the median's uncertainty is less conveniently
summarized by an elementary formula than the sample mean's.

------------------------------------------------------------------------

# 15. Vectorized bootstrap sampling

NumPy can generate many bootstrap samples at once.

``` python
samples = rng.choice(
    x,
    size=(B, len(x)),
    replace=True
)
```

Then:

``` python
bootstrap_means = (
    samples.mean(axis=1)
)
```

This is a direct application of Weeks 9--10:

``` text
array shape
+
vectorization
+
repeated simulation
```

------------------------------------------------------------------------

# 16. Memory vs. speed

Vectorization can be fast, but:

``` python
size=(1_000_000, 1000)
```

creates a very large array.

Therefore:

> **The most vectorized approach is not automatically the best
> computational approach.**

Sometimes batching or a loop is more memory-efficient.

------------------------------------------------------------------------

# 17. Resample units, not arbitrary cells

Suppose each row represents one participant with several variables.

A bootstrap sample should generally sample **participants/rows**,
preserving the variables that belong together.

Incorrect concept:

``` text
resample age column independently
resample score column independently
```

This destroys the joint relationships between variables.

Resampling must respect the data-generating unit.

------------------------------------------------------------------------

# 18. Bootstrap paired observations

Suppose data are:

``` text
x_i, y_i
```

for each participant.

To bootstrap the correlation, sample row indices:

``` python
indices = rng.choice(
    len(x),
    size=len(x),
    replace=True
)

x_b = x[indices]
y_b = y[indices]
```

The same indices preserve pairings.

------------------------------------------------------------------------

# 19. Permutation tests

Permutation testing asks a different question.

Suppose two groups have observed outcomes:

``` python
group_a
group_b
```

Observed statistic:

``` python
observed_difference = (
    group_a.mean()
    - group_b.mean()
)
```

We want to know:

> How unusual is this difference if group labels are exchangeable under
> the null hypothesis?

------------------------------------------------------------------------

# 20. Exchangeability under the null

For a simple independent two-group permutation test, the null hypothesis
implies that group labels can be rearranged without changing the joint
distribution.

Conceptually:

``` text
Observed values
       +
Observed group labels
       ↓
Under H0, labels are arbitrary
       ↓
Shuffle labels
       ↓
Recalculate statistic
```

This generates the null distribution.

SciPy describes the independent-sample permutation null as one in which
observations are pooled and randomly reassigned to samples under the
hypothesis that the samples come from the same underlying distribution.

------------------------------------------------------------------------

# 21. One permutation replicate

Pool:

``` python
combined = np.concatenate([
    group_a,
    group_b
])
```

Shuffle:

``` python
permuted = rng.permutation(
    combined
)
```

NumPy's generator `permutation()` returns a randomly permuted copy of an
array.

Split:

``` python
a_perm = permuted[
    :len(group_a)
]

b_perm = permuted[
    len(group_a):
]
```

Calculate:

``` python
difference_perm = (
    a_perm.mean()
    - b_perm.mean()
)
```

------------------------------------------------------------------------

# 22. The permutation null distribution

Repeat:

``` python
B = 10000

null_distribution = np.empty(B)

for b in range(B):
    permuted = rng.permutation(
        combined
    )

    a_perm = permuted[
        :len(group_a)
    ]

    b_perm = permuted[
        len(group_a):
    ]

    null_distribution[b] = (
        a_perm.mean()
        - b_perm.mean()
    )
```

The resulting distribution represents values of the statistic expected
under the null randomization mechanism.

------------------------------------------------------------------------

# 23. Observed vs. null statistic

Now compare:

``` text
observed statistic
```

with:

``` text
permutation null distribution
```

If the observed statistic lies deep in a tail, it is unusual under the
null hypothesis.

This is the computational basis for the permutation p-value.

------------------------------------------------------------------------

# 24. One-sided p-value

Suppose:

``` text
H₁: μ_A - μ_B > 0
```

Then estimate:

``` python
p_value = (
    null_distribution
    >= observed_difference
).mean()
```

This measures the fraction of permutation statistics at least as large
as the observed statistic.

------------------------------------------------------------------------

# 25. Two-sided p-values

For a symmetric difference statistic, a common computational definition
is:

``` python
p_value = (
    np.abs(
        null_distribution
    )
    >= np.abs(
        observed_difference
    )
).mean()
```

This counts null statistics at least as extreme in absolute value.

Two-sided definitions can require care for asymmetric statistics or
distributions.

------------------------------------------------------------------------

# 26. Monte Carlo permutation p-values

If only a random subset of all possible permutations is used, the
p-value is itself a Monte Carlo estimate.

A common correction is:

``` python
p_value = (
    1
    + np.sum(
        np.abs(null_distribution)
        >= np.abs(observed_difference)
    )
) / (
    B + 1
)
```

This avoids reporting a Monte Carlo p-value of exactly zero.

------------------------------------------------------------------------

# 27. Exact vs. random permutation tests

For very small datasets, it may be possible to enumerate every distinct
reassignment.

Then:

``` text
exact permutation distribution
```

can be calculated.

For larger datasets, the number of possible assignments becomes
enormous.

SciPy switches to exact calculation when the requested number of
resamples is at least the number of distinct permutations; otherwise it
approximates the null distribution with random resamples.

------------------------------------------------------------------------

# 28. Permutation tests depend on the null mechanism

Not every permutation means the same thing.

Possible designs include:

``` text
independent groups
paired samples
paired relationships/correlations
```

The valid permutation must preserve the experimental/data structure.

For paired data, randomly shuffling all observations as though they were
independent may be invalid.

------------------------------------------------------------------------

# 29. Paired permutation logic

Suppose each participant has:

``` text
before
after
```

One simple paired null randomization can work with differences:

``` python
difference = after - before
```

Under an appropriate symmetry null, randomly flip signs:

``` text
+d_i
or
-d_i
```

This preserves pairing.

The design determines the resampling method.

------------------------------------------------------------------------

# 30. Bootstrap vs. permutation: same mechanics, different worlds

Both use:

``` text
repeat
resample
calculate statistic
build empirical distribution
```

But:

### Bootstrap

Creates a stand-in for repeated sampling from the population.

### Permutation

Creates a world in which the null hypothesis/randomization structure
holds.

This distinction is more important than the surface similarity of the
code.

------------------------------------------------------------------------

# 31. Resampling distributions are conditional on the observed data

Both procedures depend on the dataset actually observed.

The bootstrap empirical population is built from those observations.

The permutation distribution rearranges those observations.

Therefore:

> Resampling cannot rescue a fundamentally unrepresentative or badly
> collected dataset.

Computational sophistication does not replace sound study design.

------------------------------------------------------------------------

# 32. Number of resamples

Larger `B` gives smoother and more stable empirical distributions.

But computational cost increases.

Useful questions:

-   Is the estimated SE stable?
-   Is the CI endpoint stable?
-   Is the p-value near a decision threshold?
-   What is the Monte Carlo variability?

As in Week 10, simulation size should be justified rather than chosen
ritualistically.

------------------------------------------------------------------------

# 33. Reproducibility

Always control randomness:

``` python
rng = np.random.default_rng(556)
```

And make resampling configuration explicit:

``` python
N_BOOT = 10000
N_PERM = 10000
```

This creates a reproducible computational experiment.

------------------------------------------------------------------------

# 34. Encapsulate resampling in functions

Example:

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

This uses a statistic function as an argument, connecting directly to
Week 7's higher-order functions.

------------------------------------------------------------------------

# 35. Generic permutation function

Likewise:

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

can be passed into a permutation routine.

This separates:

``` text
resampling mechanism
```

from:

``` text
statistic being evaluated
```

That is good software design and good statistical design.

------------------------------------------------------------------------

# 36. Validate against trusted implementations

After implementing a method from scratch, compare with a library
implementation.

SciPy currently provides:

``` python
scipy.stats.bootstrap
```

and:

``` python
scipy.stats.permutation_test
```

SciPy's permutation interface accepts a statistic function, resampling
count, alternative hypothesis, permutation type, and RNG.

The pedagogical sequence should be:

``` text
understand
   ↓
implement
   ↓
validate
   ↓
use library appropriately
```

not simply:

``` text
call function
```

------------------------------------------------------------------------

# 37. Common mistakes

## Bootstrap mistake 1

Sampling without replacement.

That simply rearranges the original data.

## Bootstrap mistake 2

Resampling columns independently when rows are the observational units.

## Bootstrap mistake 3

Interpreting a bootstrap distribution as though it were the
observed-data distribution.

## Permutation mistake 1

Ignoring paired/clustered study design.

## Permutation mistake 2

Permuting when exchangeability under the null is not plausible.

## Permutation mistake 3

Using the bootstrap distribution as the null distribution.

------------------------------------------------------------------------

# 38. A resampling workflow

``` text
Define statistical question
        ↓
Choose statistic
        ↓
Identify observational unit
        ↓
Choose valid resampling mechanism
        ↓
Generate empirical distribution
        ↓
Quantify uncertainty / extremeness
        ↓
Check Monte Carlo stability
        ↓
Interpret statistically
```

------------------------------------------------------------------------

# 39. Key ideas

By the end of Week 11, you should be able to explain:

1.  What resampling means.
2.  Why bootstrap samples use replacement.
3.  What the empirical distribution represents.
4.  What a bootstrap distribution approximates.
5.  How to calculate a bootstrap standard error.
6.  How a percentile bootstrap interval is constructed.
7.  How to bootstrap statistics other than the mean.
8.  Why resampling should preserve observational units.
9.  What exchangeability means in permutation testing.
10. How to construct a two-group permutation null distribution.
11. How to calculate one- and two-sided permutation p-values.
12. Why random permutation p-values have Monte Carlo uncertainty.
13. Exact vs. Monte Carlo permutation tests.
14. Why paired designs require different randomization schemes.
15. The conceptual difference between bootstrap and permutation.
16. Why resampling does not repair poor study design.
17. How Weeks 7, 9, and 10 support reusable resampling implementations.
18. Why implementations should be validated against trusted libraries.

------------------------------------------------------------------------

# 40. Recommended reading

## SciPy --- Bootstrap

The SciPy statistics documentation provides a production implementation
of bootstrap confidence intervals. After implementing bootstrap logic
from scratch, compare your design with `scipy.stats.bootstrap`.

https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.bootstrap.html

## SciPy --- Permutation Test

The official SciPy reference explains independent, paired-sample, and
pairing permutation schemes, exact vs. randomized testing, null
distributions, and p-values.

https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.permutation_test.html

## NumPy --- Generator.choice

Useful reference for sampling with replacement.

https://numpy.org/doc/stable/reference/random/generated/numpy.random.Generator.choice.html

## NumPy --- Generator.permutation

Reference for generating random permutations.

https://numpy.org/doc/stable/reference/random/generated/numpy.random.Generator.permutation.html

------------------------------------------------------------------------

# 41. YouTube recommendations

## 1. Emma Ding --- "What are Bootstrap and Permutation Tests in Data Science?"

A compact conceptual comparison of resampling, bootstrap distributions,
permutation tests, and when the two procedures answer different
statistical questions.

**Recommended use:** Watch before the hands-on tutorial. It gives
students the conceptual distinction before they implement either method.

[Watch on YouTube](https://www.youtube.com/watch?v=uGsf3spCM3Y)

------------------------------------------------------------------------

## 2. StatQuest --- Bootstrapping

StatQuest's bootstrapping material provides a visual explanation of
sampling with replacement and using repeated resamples to characterize
estimator uncertainty.

**Recommended use:** Watch alongside the bootstrap-distribution and
confidence-interval sections.

[Find on
YouTube](https://www.youtube.com/results?search_query=StatQuest+bootstrapping)

------------------------------------------------------------------------

## 3. Permutation Tests / Randomization Tests

A visual permutation-test tutorial can reinforce the key idea that
labels are rearranged to construct the distribution expected under the
null hypothesis.

**Recommended use:** Watch before implementing the two-group permutation
test.

[Find on
YouTube](https://www.youtube.com/results?search_query=permutation+test+randomization+test+statistics+tutorial)

------------------------------------------------------------------------

# 42. Week 11 takeaway

The central lesson is:

> **Resampling constructs useful reference distributions by repeatedly
> reorganizing the information contained in the observed data.**

``` text
Observed sample
      ↓
Choose statistic
      ↓
Bootstrap?
   sample with replacement
      ↓
sampling-distribution approximation

or

Permutation?
   rearrange under H0
      ↓
null-distribution approximation
      ↓
SE / CI / p-value
```

Next week we move into **numerical optimization**, including root
finding, Newton--Raphson, gradient descent, and statistical objective
functions.
