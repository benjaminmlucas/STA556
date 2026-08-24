# 7.2 Activity

## Profiling, Bottlenecks, Parallelism & Numba

**Tools:** Python, NumPy, pandas, pytest, cProfile, optional
line_profiler, optional Numba, Jupyter/VS Code

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

-   benchmark Python code;
-   distinguish noisy one-off timings from repeated benchmarks;
-   identify algorithmic bottlenecks;
-   use `cProfile`;
-   optionally use line-level profiling;
-   inspect memory consumption;
-   compare loop and vectorized implementations;
-   reason about vectorization vs. memory tradeoffs;
-   benchmark chunked processing;
-   understand parallel overhead;
-   accelerate appropriate loops with Numba;
-   separate JIT compilation time from steady-state timing;
-   validate optimized code against tests.

------------------------------------------------------------------------

# Part 0 --- Set up

Create:

``` text
notebooks/week14_performance.ipynb
```

and:

``` text
src/week14_performance.py
```

Import:

``` python
from time import perf_counter

import numpy as np
import pandas as pd
```

Create:

``` python
rng = np.random.default_rng(
    556
)
```

Before optimizing anything, run your Week 13 test suite:

``` bash
pytest
```

------------------------------------------------------------------------

## Codespaces performance note

All performance comparisons in this activity should be made **within your own Codespace**. Do not interpret differences between students' absolute timings as differences in code quality. Cloud machine size and system load may differ.

For multiprocessing exercises, first inspect the CPU resources visible to your environment:

```python
import os

os.cpu_count()
```

Parallel code is not expected to be faster for every workload or every Codespace. Measuring the overhead is part of the lesson.

---

# Part 1 --- One-off timing

Create:

``` python
x = rng.normal(
    size=1_000_000
)
```

Time:

``` python
start = perf_counter()

result = np.sum(
    x ** 2
)

elapsed = (
    perf_counter()
    - start
)

elapsed
```

Repeat manually several times.

### Question

Are all timings identical?

Why not?

------------------------------------------------------------------------

# Part 2 --- Use `%timeit`

Run:

``` python
%timeit np.sum(
    x ** 2
)
```

Compare with your one-off timings.

### Reflection

Why is repeated benchmarking usually more informative?

------------------------------------------------------------------------

# Part 3 --- Python loop vs. NumPy

Write:

``` python
def sum_squares_python(
    x
):
    total = 0.0

    for value in x:
        total += (
            value
            * value
        )

    return total
```

Compare with:

``` python
def sum_squares_numpy(
    x
):
    return np.sum(
        x ** 2
    )
```

Verify:

``` python
np.isclose(
    sum_squares_python(x),
    sum_squares_numpy(x)
)
```

Then benchmark both.

------------------------------------------------------------------------

# Part 4 --- Correctness before speed

Temporarily introduce a bug:

``` python
total += value
```

instead of:

``` python
total += value * value
```

The broken version may still be fast.

### Question

Why must equivalent output be verified before benchmarking
implementations?

Restore the correct code.

------------------------------------------------------------------------

# Part 5 --- Scaling experiment

Benchmark both methods using:

``` text
n = 1,000
10,000
100,000
1,000,000
```

Store:

``` text
n
python_time
numpy_time
speedup
```

in a DataFrame.

### Reflection

Does the relative advantage stay constant as `n` changes?

------------------------------------------------------------------------

# Part 6 --- Algorithmic bottleneck

Create:

``` python
values = list(
    range(
        100_000
    )
)

lookup_values = list(
    range(
        50_000,
        150_000
    )
)
```

Write:

``` python
def count_matches_list(
    values,
    lookup_values
):
    count = 0

    for value in values:
        if value in lookup_values:
            count += 1

    return count
```

Benchmark.

------------------------------------------------------------------------

# Part 7 --- Change the data structure

Rewrite:

``` python
def count_matches_set(
    values,
    lookup_values
):
    lookup = set(
        lookup_values
    )

    count = 0

    for value in values:
        if value in lookup:
            count += 1

    return count
```

Verify identical output.

Benchmark.

### Question

Was the improvement caused by syntax or algorithm/data structure?

------------------------------------------------------------------------

# Part 8 --- Profile a workflow

Add to `src/week14_performance.py`:

``` python
def slow_transform(
    x
):
    output = []

    for value in x:
        output.append(
            np.sqrt(
                value ** 2
                + 1
            )
        )

    return np.array(
        output
    )


def summarize(
    x
):
    transformed = (
        slow_transform(
            x
        )
    )

    return {
        "mean": transformed.mean(),
        "sd": transformed.std(),
        "max": transformed.max()
    }


def main():
    rng = np.random.default_rng(
        556
    )

    x = rng.normal(
        size=500_000
    )

    print(
        summarize(x)
    )


if __name__ == "__main__":
    main()
```

------------------------------------------------------------------------

# Part 9 --- Use cProfile

From the terminal:

``` bash
python -m cProfile \
    src/week14_performance.py
```

Inspect the output.

Identify:

``` text
which function has highest cumulative time?
which has highest direct time?
how many calls were made?
```

------------------------------------------------------------------------

# Part 10 --- Sort profile output

Use:

``` bash
python -m cProfile \
    -s cumulative \
    src/week14_performance.py
```

### Question

Why is sorting by cumulative time useful when investigating a call
chain?

------------------------------------------------------------------------

# Part 11 --- Profile only the hotspot

Once `slow_transform()` is identified, focus your attention there.

### Reflection

Why would optimizing `summarize()` first be a poor use of time if almost
all runtime occurs inside `slow_transform()`?

------------------------------------------------------------------------

# Part 12 --- Optional line profiling

If installed:

``` python
%load_ext line_profiler
```

Import the function, then run:

``` python
%lprun \
    -f slow_transform \
    slow_transform(x)
```

Identify the line consuming most runtime.

If `line_profiler` is unavailable, use this as a conceptual exercise
rather than installing blindly.

------------------------------------------------------------------------

# Part 13 --- Vectorize the hotspot

Rewrite:

``` python
def fast_transform(
    x
):
    return np.sqrt(
        x ** 2
        + 1
    )
```

Check:

``` python
slow = slow_transform(
    x[:1000]
)

fast = fast_transform(
    x[:1000]
)

np.allclose(
    slow,
    fast
)
```

Benchmark both on a large array.

------------------------------------------------------------------------

# Part 14 --- Reprofile

Replace the slow implementation in the workflow.

Run `cProfile` again.

### Questions

1.  What is now the bottleneck?
2.  How did total runtime change?
3.  Why must profiling be repeated after optimization?

------------------------------------------------------------------------

# Part 15 --- Inspect memory use

For:

``` python
x = rng.normal(
    size=10_000_000
)
```

Inspect:

``` python
x.nbytes
```

Convert to MB:

``` python
x.nbytes / (
    1024 ** 2
)
```

### Question

How much memory do 10 million `float64` values require?

------------------------------------------------------------------------

# Part 16 --- DataFrame memory

Create:

``` python
df = pd.DataFrame({
    "id": np.arange(
        1_000_000
    ),
    "group": rng.choice(
        [
            "A",
            "B",
            "C"
        ],
        size=1_000_000
    ),
    "score": rng.normal(
        size=1_000_000
    )
})
```

Inspect:

``` python
df.memory_usage(
    deep=True
)
```

and:

``` python
df.memory_usage(
    deep=True
).sum()
```

------------------------------------------------------------------------

# Part 17 --- Efficient dtypes

Convert:

``` python
df["group"] = (
    df["group"]
    .astype(
        "category"
    )
)
```

Recheck memory.

### Reflection

Why can categorical data use less memory than repeated Python strings?

------------------------------------------------------------------------

# Part 18 --- Copying cost

Run:

``` python
df_copy = df.copy()
```

Estimate the additional memory footprint.

### Question

When is copying worth the memory cost?

Connect this to Week 2's mutation/reference material.

------------------------------------------------------------------------

# Part 19 --- Select only needed columns

Compare:

``` python
large_subset = df.copy()
```

with:

``` python
small_subset = df[
    [
        "group",
        "score"
    ]
].copy()
```

Compare memory usage.

### Principle

Move and retain only the data needed for the task.

------------------------------------------------------------------------

# Part 20 --- Chunking concept

Create a temporary CSV from a manageable DataFrame.

Then read:

``` python
for chunk in pd.read_csv(
    path,
    chunksize=100_000
):
    ...
```

Calculate the sum/count required for an overall mean without loading the
entire file at once.

### Challenge

Maintain:

``` text
running_sum
running_count
```

then calculate:

``` text
running_sum / running_count
```

------------------------------------------------------------------------

# Part 21 --- Verify chunked output

Compare the chunked result with:

``` python
pd.read_csv(
    path
)["score"].mean()
```

Use:

``` python
np.isclose(...)
```

### Question

What resource tradeoff does chunking make?

------------------------------------------------------------------------

# Part 22 --- Parallel-task concept

Suppose:

``` python
def simulation_scenario(
    seed
):
    rng = np.random.default_rng(
        seed
    )

    return rng.normal(
        size=1_000_000
    ).mean()
```

Each seed defines an independent task.

### Question

Why is this a good candidate for parallel execution?

------------------------------------------------------------------------

# Part 23 --- Serial baseline

Create several independent seeds.

Run each scenario serially.

Time total execution.

Store results.

This is your baseline.

------------------------------------------------------------------------

# Part 24 --- Process-based parallelism

Use Python's standard library:

``` python
from concurrent.futures import (
    ProcessPoolExecutor
)
```

Conceptually:

``` python
with ProcessPoolExecutor() as executor:
    results = list(
        executor.map(
            simulation_scenario,
            seeds
        )
    )
```

Benchmark.

### Reflection

Did parallelism help?

If not, what overhead might explain the result?

------------------------------------------------------------------------

# Part 25 --- Vary task size

Repeat the serial/parallel comparison for:

``` text
small per-task workload
large per-task workload
```

### Question

At what point does parallel overhead become less important?

------------------------------------------------------------------------

# Part 26 --- Randomness and parallel workers

Do **not** use:

``` python
np.random.default_rng(
    556
)
```

inside every worker with the same seed.

Instead, pass distinct reproducible seeds.

### Question

What statistical error could identical random streams create?

------------------------------------------------------------------------

# Part 27 --- Install/check Numba

Check:

``` python
import numba

numba.__version__
```

If Numba is not installed, the remaining Numba sections may be treated
as optional.

The goal is to understand when JIT compilation helps.

------------------------------------------------------------------------

# Part 28 --- Numba baseline loop

Use:

``` python
def sum_squares_loop(
    x
):
    total = 0.0

    for value in x:
        total += (
            value
            * value
        )

    return total
```

Benchmark.

------------------------------------------------------------------------

# Part 29 --- Compile with `@njit`

If Numba is available:

``` python
from numba import njit
```

Create:

``` python
@njit
def sum_squares_numba(
    x
):
    total = 0.0

    for value in x:
        total += (
            value
            * value
        )

    return total
```

Call once:

``` python
sum_squares_numba(
    x
)
```

Then benchmark subsequent calls.

------------------------------------------------------------------------

# Part 30 --- Compilation overhead

Time:

``` text
first Numba call
```

and:

``` text
second Numba call
```

separately.

### Reflection

Why should the first call not be directly compared with steady-state
NumPy execution?

------------------------------------------------------------------------

# Part 31 --- Compare Python, NumPy, and Numba

Benchmark:

``` text
pure Python loop
NumPy vectorized
Numba-compiled loop
```

Verify all results are approximately equal.

Create a table:

``` text
method
runtime
speedup vs Python
```

------------------------------------------------------------------------

# Part 32 --- When NumPy is already excellent

Compare a matrix multiplication:

``` python
A @ B
```

with any hand-written loop version.

### Question

Why might Numba wrapping add little to an operation already implemented
through optimized numerical libraries?

------------------------------------------------------------------------

# Part 33 --- Numba-friendly algorithm challenge

Create a numerical loop that is less convenient to vectorize.

Example: a simple recurrence.

``` python
def recurrence(
    x
):
    output = np.empty_like(
        x
    )

    output[0] = x[0]

    for i in range(
        1,
        len(x)
    ):
        output[i] = (
            0.9
            * output[i - 1]
            + x[i]
        )

    return output
```

Compile with Numba and compare.

### Question

Why is this a better Numba candidate than simple element-wise squaring?

------------------------------------------------------------------------

# Part 34 --- Optional Numba parallel loop

If supported, experiment with:

``` python
from numba import (
    njit,
    prange
)
```

Use:

``` python
@njit(
    parallel=True
)
def ...
```

with an independent loop.

Compare serial-JIT and parallel-JIT performance.

### Important

Verify correctness before interpreting speed.

------------------------------------------------------------------------

# Part 35 --- Performance regression check

After every optimization, rerun:

``` bash
pytest
```

### Question

Why is Week 13's test suite essential when replacing a simple
implementation with a faster one?

------------------------------------------------------------------------

# Part 36 --- Mini optimization project

Choose one previous course algorithm:

``` text
bootstrap
permutation test
Monte Carlo integration
gradient descent
distance calculation
```

Complete:

1.  Run tests / create correctness checks.
2.  Benchmark a representative workload.
3.  Profile it.
4.  Identify the hotspot.
5.  Make exactly one major performance improvement.
6.  Re-run correctness tests.
7.  Rebenchmark.
8.  Calculate speedup.
9.  Discuss memory implications.
10. Explain whether the added complexity is worth it.

------------------------------------------------------------------------

# Part 37 --- Amdahl's-law thought experiment

Suppose:

``` text
90 seconds
```

of a:

``` text
100-second
```

program occurs inside one function.

If you make that function 10 times faster:

``` text
90 sec → 9 sec
```

What is the new total runtime?

What is the total speedup?

### Reflection

Why is hotspot identification so powerful?

------------------------------------------------------------------------

# Part 38 --- Performance audit

For your mini project, answer:

``` text
What is the representative input size?
What was the baseline runtime?
How was runtime measured?
What did profiling identify?
What optimization was applied?
Did memory usage change?
Did tests still pass?
What is the new runtime?
What is the speedup?
What new complexity was introduced?
Would you keep the optimized version?
```

------------------------------------------------------------------------

# Part 39 --- Git checkpoint

Run:

``` bash
pytest
```

Then:

``` bash
git status
git add .
git commit -m "Complete Week 14 profiling and performance exercises"
git push
```

------------------------------------------------------------------------

# Part 40 --- Final reflection

### 1. Profiling

Why should profiling occur before optimization?

### 2. Benchmarking

Why are repeated representative benchmarks preferable to one timing?

### 3. Complexity

Why can changing an algorithm or data structure outperform
micro-optimization?

### 4. Vectorization

Why is vectorization often faster?

### 5. Memory

Why can a fast vectorized method still be inappropriate?

### 6. Parallelism

When can parallel computation be slower than serial computation?

### 7. RNG

Why do parallel simulations need independent random streams?

### 8. Numba

What does JIT compilation do?

### 9. Compilation overhead

Why separate first-call and steady-state Numba timing?

### 10. Software engineering

Why must performance optimization be followed by correctness testing?

------------------------------------------------------------------------

# Completion checklist

-   [ ] Created Week 14 notebook
-   [ ] Ran correctness tests before optimization
-   [ ] Timed code using `perf_counter`
-   [ ] Used `%timeit`
-   [ ] Compared Python-loop and NumPy implementations
-   [ ] Verified equivalent output before benchmarking
-   [ ] Ran a scaling experiment
-   [ ] Improved a membership algorithm using a better data structure
-   [ ] Created a profileable Python script
-   [ ] Used `cProfile`
-   [ ] Interpreted cumulative vs. direct timing
-   [ ] Reprofiled after optimization
-   [ ] Used line profiling or reviewed the concept
-   [ ] Inspected NumPy array memory
-   [ ] Inspected pandas memory usage
-   [ ] Reduced memory with an efficient dtype
-   [ ] Investigated copying cost
-   [ ] Selected only needed columns
-   [ ] Processed data in chunks
-   [ ] Validated chunked output
-   [ ] Established a serial parallel-computing baseline
-   [ ] Used or reviewed `ProcessPoolExecutor`
-   [ ] Compared parallel workloads of different sizes
-   [ ] Used distinct reproducible RNG seeds
-   [ ] Checked Numba availability
-   [ ] Compiled a numerical loop with `@njit`
-   [ ] Separated compilation from execution timing
-   [ ] Compared Python, NumPy, and Numba
-   [ ] Identified a case where NumPy is already optimized
-   [ ] Accelerated a recurrence-like loop with Numba
-   [ ] Explored `prange` or reviewed parallel JIT concepts
-   [ ] Reran tests after optimization
-   [ ] Completed the mini optimization project
-   [ ] Calculated overall speedup
-   [ ] Audited the performance tradeoff
-   [ ] Committed Week 14 work to Git
-   [ ] Pushed work to GitHub

------------------------------------------------------------------------

# What you should now understand

``` text
Correct code
    ↓
Benchmark
    ↓
Profile
    ↓
Find hotspot
    ↓
Improve algorithm / vectorize / reduce memory / parallelize / JIT
    ↓
Retest
    ↓
Rebenchmark
    ↓
Keep only worthwhile complexity
```

This completes the main computational workflow arc of STA 556: from
environments and data engineering through reproducibility, numerical
statistics, testing, and performance-aware statistical software.
