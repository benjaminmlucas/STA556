# 7.2 Profiling, Performance & Acceleration

## Why this week matters

By Week 14, you have written code that can:

-   manipulate complex data;
-   simulate large numbers of observations;
-   bootstrap and permute;
-   solve numerical optimization problems;
-   validate results with automated tests.

The next question is:

> **When code is too slow or uses too much memory, how do we improve it
> without sacrificing correctness or readability?**

This week focuses on:

-   timing code correctly;
-   identifying bottlenecks;
-   profiling function and line-level performance;
-   understanding memory use;
-   improving algorithms before micro-optimizing;
-   vectorization;
-   avoiding unnecessary copies;
-   parallel computation;
-   Numba JIT compilation;
-   performance validation.

The core principle is:

> **Measure first. Optimize second.**

------------------------------------------------------------------------

# 1. Performance is multidimensional

Performance can mean:

``` text
runtime
memory usage
scalability
latency
throughput
```

A change that improves one may worsen another.

For example:

``` text
vectorize everything
```

may reduce runtime but create a huge temporary array and increase memory
use.

Optimization is a tradeoff, not a single number.

------------------------------------------------------------------------

# 2. Correctness comes first

Fast incorrect code is still incorrect.

A useful order is:

``` text
correct
  ↓
tested
  ↓
profiled
  ↓
optimized
  ↓
retested
```

Week 13's test suite becomes especially important this week.

Before changing an implementation for speed, make sure tests capture its
intended behavior.

------------------------------------------------------------------------

# 3. Do not optimize by intuition alone

Humans are often poor at guessing which line dominates runtime.

A function may look slow because it contains:

``` python
for ...
```

while the actual bottleneck may be:

``` python
large_matrix @ large_matrix
```

or repeated file I/O.

Therefore:

> **Profiling is empirical debugging for performance.**

------------------------------------------------------------------------

# 4. Benchmarking small expressions

In Jupyter:

``` python
%timeit
```

is useful for repeated timing.

Example:

``` python
%timeit np.sum(x)
```

versus:

``` python
%timeit sum(x)
```

`%timeit` repeats the operation enough times to estimate typical
execution speed rather than relying on one potentially noisy run.

------------------------------------------------------------------------

# 5. Why one timing is unreliable

Runtime varies because of:

-   operating-system scheduling;
-   caching;
-   background processes;
-   CPU frequency;
-   memory allocation;
-   first-run compilation;
-   disk/network variability.

A single timing may not represent typical performance.

Repeated measurements are preferable.

------------------------------------------------------------------------

# 6. `time.perf_counter()`

For larger blocks:

``` python
from time import perf_counter

start = perf_counter()

result = expensive_function()

elapsed = (
    perf_counter()
    - start
)

print(elapsed)
```

`perf_counter()` is appropriate for measuring elapsed wall-clock time.

------------------------------------------------------------------------

# 7. Benchmark representative workloads

Do not benchmark only tiny toy inputs if production inputs are large.

An algorithm may be:

``` text
fast for n = 100
slow for n = 1,000,000
```

Performance depends on problem scale.

Benchmark at realistic sizes.

------------------------------------------------------------------------

# 8. Algorithmic complexity

Before optimizing syntax, ask whether the algorithm itself is
inefficient.

Examples:

``` text
O(n)
O(n log n)
O(n²)
O(n³)
```

If runtime doubles approximately when `n` doubles:

``` text
possibly O(n)
```

If runtime grows by roughly four:

``` text
possibly O(n²)
```

Improving the algorithm often matters much more than replacing one
Python expression with another.

------------------------------------------------------------------------

# 9. Example: repeated membership checking

Slow pattern:

``` python
for x in values:
    if x in large_list:
        ...
```

Membership in a list may require linear scanning.

Using a set:

``` python
lookup = set(
    large_list
)
```

can make repeated membership tests much faster.

This is an algorithm/data-structure improvement, not a
micro-optimization.

------------------------------------------------------------------------

# 10. Profiling with `cProfile`

Python includes:

``` python
cProfile
```

for function-level profiling.

From the terminal:

``` bash
python -m cProfile script.py
```

Or in Python:

``` python
import cProfile

cProfile.run(
    "main()"
)
```

The profiler records where execution time is spent across function
calls.

------------------------------------------------------------------------

# 11. Interpreting profile output

Typical columns include:

``` text
ncalls
tottime
percall
cumtime
```

Conceptually:

``` text
tottime
→ time spent directly inside function

cumtime
→ function time + functions it calls
```

A high cumulative time can indicate an important bottleneck higher in
the call tree.

------------------------------------------------------------------------

# 12. Profile before changing code

A good workflow:

``` text
run representative workload
      ↓
profile
      ↓
identify top hotspot
      ↓
optimize one bottleneck
      ↓
profile again
```

Do not rewrite the entire program at once.

------------------------------------------------------------------------

# 13. Line-level profiling

Function profiling may reveal:

``` text
slow_function()
```

but not which line is expensive.

`line_profiler` provides line-by-line timing for selected functions and
is particularly useful for scientific Python code where one NumPy
expression may dominate execution. citeturn174684search0

Typical Jupyter usage:

``` python
%load_ext line_profiler
```

Then:

``` python
%lprun -f slow_function slow_function(...)
```

------------------------------------------------------------------------

# 14. Memory matters

A program may be slow because it allocates too much memory.

Potential symptoms:

-   swapping;
-   process termination;
-   unexpectedly large temporary arrays;
-   repeated DataFrame copies;
-   inability to scale to realistic data.

Runtime and memory profiling should be considered together.

------------------------------------------------------------------------

# 15. Estimate array memory

For NumPy:

``` python
x.nbytes
```

For a DataFrame:

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

Understanding object size helps explain performance behavior.

------------------------------------------------------------------------

# 16. Temporary arrays

Consider:

``` python
z = (
    (x - mean) ** 2
    / sd
)
```

This may create intermediate arrays.

For moderate data, that is fine.

For enormous arrays, temporary allocations may matter.

Optimization should be driven by actual measurements, not premature fear
of temporary objects.

------------------------------------------------------------------------

# 17. Avoid unnecessary copies

In pandas:

``` python
copy = df.copy()
```

duplicates data intentionally.

That may be necessary for correctness, but copying a very large
DataFrame repeatedly can be expensive.

Ask:

``` text
Do I need a copy?
Am I modifying the original?
Can I select fewer columns?
Can I aggregate earlier?
```

------------------------------------------------------------------------

# 18. Push work toward the source

Week 5 introduced this principle for SQL.

Instead of:

``` text
load 100 million rows
then filter to 50,000
```

prefer:

``` text
filter in SQL
then transfer 50,000 rows
```

Performance optimization often means **moving less data**.

------------------------------------------------------------------------

# 19. Vectorization revisited

Week 9 showed:

``` python
result = x ** 2
```

instead of:

``` python
result = []

for value in x:
    result.append(
        value ** 2
    )
```

Vectorization moves iteration into optimized numerical code.

This is often one of the highest-value performance improvements in
scientific Python.

------------------------------------------------------------------------

# 20. Vectorization can also be overused

Suppose vectorization requires creating a matrix of shape:

``` text
1,000,000 × 10,000
```

That may be impossible in memory.

An explicit loop over chunks may be better.

Thus:

> **Vectorization is a tool, not a rule.**

------------------------------------------------------------------------

# 21. Chunking

For large data:

``` text
whole dataset
     ↓
chunk 1
chunk 2
chunk 3
...
```

Process each piece separately.

Example:

``` python
for chunk in pd.read_csv(
    "large.csv",
    chunksize=100_000
):
    process(
        chunk
    )
```

Chunking trades some simplicity for lower peak memory.

------------------------------------------------------------------------

# 22. Parallelism

If independent tasks can be performed simultaneously, parallel execution
may reduce elapsed time.

Examples:

``` text
independent bootstrap replications
simulation scenarios
processing separate files
cross-validation folds
```

But parallelism has overhead.

------------------------------------------------------------------------

# 23. Parallel overhead

Parallel computation requires:

-   starting/maintaining workers;
-   serializing data;
-   transferring data;
-   collecting results;
-   scheduling tasks.

For tiny tasks:

``` text
parallel version
```

may be slower than:

``` text
serial version
```

Parallelism is beneficial when per-task work is large enough to outweigh
overhead.

------------------------------------------------------------------------

# 24. Threads vs. processes

At a high level:

### Threads

Share memory within one process.

Useful for many I/O-bound tasks and some native-code numerical
workloads.

### Processes

Use separate Python processes.

Can bypass Python's Global Interpreter Lock for CPU-bound pure Python
work, but require data serialization and separate memory.

The right choice depends on workload.

------------------------------------------------------------------------

# 25. Embarrassingly parallel tasks

A task is often called **embarrassingly parallel** when pieces require
little or no communication.

Example:

``` text
simulate scenario 1
simulate scenario 2
simulate scenario 3
...
```

Each scenario can run independently.

These are good candidates for parallel computation.

------------------------------------------------------------------------

# 26. Reproducible RNG in parallel

Parallel simulation requires care.

Do not give every worker the same seed.

Otherwise workers may generate identical streams.

A robust design uses independent random streams derived from a
reproducible seed strategy.

Random-number design is part of parallel statistical computing.

------------------------------------------------------------------------

# 27. Numba

Numba is a JIT compiler for numerical Python.

The current Numba documentation provides tools such as:

``` python
@njit
```

for compiling Python functions and:

``` python
prange
```

for supported parallel loops. citeturn174684search1

Numba can be especially effective for loops involving:

-   numerical scalars;
-   NumPy arrays;
-   repeated arithmetic;
-   algorithms difficult to vectorize cleanly.

------------------------------------------------------------------------

# 28. JIT compilation

JIT means:

``` text
Just-In-Time compilation
```

The first time a function runs for a given set of types, Numba may
compile it to machine code.

Therefore:

``` text
first call
```

may be slower.

Later calls may be much faster.

This is why Numba benchmarking must distinguish compilation time from
execution time.

------------------------------------------------------------------------

# 29. Basic Numba example

``` python
from numba import njit
```

Define:

``` python
@njit
def sum_squares(
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

The loop remains readable Python, but Numba can compile it.

------------------------------------------------------------------------

# 30. When Numba helps

Numba is often useful when:

-   pure Python loops dominate runtime;
-   NumPy vectorization is awkward;
-   data are mostly numeric;
-   the function is called repeatedly;
-   algorithmic complexity is already reasonable.

Numba is not a substitute for choosing a better algorithm.

------------------------------------------------------------------------

# 31. When Numba may not help

Potential poor candidates include code dominated by:

-   pandas operations;
-   Python objects/strings;
-   network requests;
-   file I/O;
-   already optimized BLAS/LAPACK calls;
-   one tiny function run only once.

Always benchmark.

------------------------------------------------------------------------

# 32. Numba and NumPy

Operations such as:

``` python
X @ beta
```

may already call highly optimized compiled linear-algebra libraries.

Adding Numba around such code may provide little benefit.

The profiler tells you where optimization effort belongs.

------------------------------------------------------------------------

# 33. Numba parallel loops

Numba supports parallel loops in suitable compiled functions.

Conceptually:

``` python
from numba import (
    njit,
    prange
)
```

``` python
@njit(
    parallel=True
)
def compute(...):
    for i in prange(n):
        ...
```

Parallel execution is not automatic magic.

The iterations must be safe to execute independently.

------------------------------------------------------------------------

# 34. Race conditions

Parallel code can be wrong if tasks update shared state unsafely.

Example concept:

``` text
worker 1 updates total
worker 2 updates total
at same time
```

The result may depend on timing.

Parallel correctness requires understanding data dependencies.

------------------------------------------------------------------------

# 35. Benchmark equivalent implementations

Suppose you implement:

``` text
Python loop
NumPy vectorized
Numba loop
Numba parallel loop
```

Compare:

-   correctness;
-   runtime;
-   memory;
-   code complexity;
-   compilation overhead.

The "fastest" version is only useful if its complexity is justified.

------------------------------------------------------------------------

# 36. Performance regression tests

Performance can regress just like correctness.

You may record benchmark results:

``` text
before optimization
after optimization
```

and document expectations.

Avoid overly strict timing assertions in normal unit tests because
shared machines have noisy timing.

Instead, use dedicated benchmarks.

------------------------------------------------------------------------

# 37. Optimization can reduce readability

Compare:

``` python
result = (
    x - x.mean()
) / x.std()
```

with a highly specialized low-level implementation.

If the readable version runs in:

``` text
0.02 seconds
```

and the complex version in:

``` text
0.01 seconds
```

the complexity may not be worthwhile.

Optimization has maintenance cost.

------------------------------------------------------------------------

# 38. Amdahl's law intuition

Suppose:

``` text
90% of runtime
```

occurs in one function.

Even if you make the other 10% infinitely fast, total runtime can
improve by only about 10%.

Therefore:

> **Optimize the dominant bottleneck, not the code that merely looks
> inefficient.**

------------------------------------------------------------------------

# 39. Performance and reproducibility

Performance decisions should be documented.

Record:

``` text
input size
hardware context
timing method
number of repetitions
software versions
```

Benchmark numbers without context are not fully reproducible.

------------------------------------------------------------------------

# 40. Performance workflow

``` text
Correct code
    ↓
Tests pass
    ↓
Representative workload
    ↓
Benchmark
    ↓
Profile
    ↓
Identify hotspot
    ↓
Choose optimization
    ↓
Retest correctness
    ↓
Rebenchmark
    ↓
Document tradeoff
```

------------------------------------------------------------------------

# 41. Common mistakes

## Mistake 1

Optimizing before measuring.

## Mistake 2

Benchmarking unrealistic toy inputs.

## Mistake 3

Comparing methods that produce different results.

## Mistake 4

Including JIT compilation time in one benchmark but not another.

## Mistake 5

Using parallelism for tiny tasks.

## Mistake 6

Ignoring memory usage.

## Mistake 7

Replacing a clear algorithm with complex code for negligible gains.

## Mistake 8

Forgetting to rerun correctness tests after optimization.

------------------------------------------------------------------------

# 42. Key ideas

By the end of Week 14, you should be able to explain:

1.  Runtime vs. memory performance.
2.  Why correctness precedes optimization.
3.  Why benchmarking should use repeated representative workloads.
4.  Algorithmic complexity.
5.  What profiling measures.
6.  Function-level vs. line-level profiling.
7.  How to inspect NumPy/DataFrame memory use.
8.  Why unnecessary data movement is expensive.
9.  The benefits and limits of vectorization.
10. Chunked processing.
11. Parallel-computing overhead.
12. Threads vs. processes conceptually.
13. Embarrassingly parallel workloads.
14. Why parallel RNG streams require care.
15. What JIT compilation means.
16. When Numba is useful.
17. Why compiled NumPy linear algebra may already be optimized.
18. What race conditions are.
19. Why readability and maintenance matter in performance work.
20. Why profiling should guide optimization priorities.

------------------------------------------------------------------------

# 43. Recommended reading

## Python --- Profilers

Python's standard library includes `profile`, `cProfile`, and `pstats`
for deterministic profiling.

https://docs.python.org/3/library/profile.html

## line_profiler

The current maintained `line_profiler` project provides line-by-line
timing for selected functions. citeturn174684search0

https://github.com/pyutils/line_profiler

## Numba Documentation

The Numba user manual covers JIT compilation, NumPy support, performance
tips, and automatic parallelization. citeturn174684search1

https://numba.readthedocs.io/en/stable/

## pandas --- Scaling to Large Datasets

Useful guidance on memory, efficient dtypes, chunking, and
larger-than-memory workflows.

https://pandas.pydata.org/docs/user_guide/scale.html

------------------------------------------------------------------------

# 44. YouTube recommendations

## 1. Real Python --- "Profiling Performance in Python: Getting Started & Benchmarking Code Snippets"

A recent introduction to the key idea of profiling before optimizing. It
discusses benchmarking and using profiling to decide whether performance
work is worth the effort. citeturn174684youtube37

**Recommended use:** Watch before beginning the profiling exercises.

[Watch on YouTube](https://www.youtube.com/watch?v=dgb0zqc0kxk)

------------------------------------------------------------------------

## 2. Misha Sv --- "I Profiled Python Code with cProfile & You Won't Believe What I Found"

A practical walkthrough of `cProfile`, including terminal use,
programmatic profiling, and exporting profiling data.
citeturn174684youtube36

**Recommended use:** Watch alongside the `cProfile` tutorial section.

[Watch on YouTube](https://www.youtube.com/watch?v=YK6MikOCEWM)

------------------------------------------------------------------------

## 3. Numba JIT Tutorials

A focused Numba tutorial is useful after students have profiled a
numerical Python loop and identified it as a genuine hotspot.

**Recommended use:** Watch only after the profiling portion so that
Numba is presented as a targeted solution rather than a default
decoration.

[Find Numba tutorials on
YouTube](https://www.youtube.com/results?search_query=Numba+njit+prange+Python+tutorial)

------------------------------------------------------------------------

# 45. Week 14 takeaway

The central lesson is:

> **Performance engineering is an evidence-based process: measure,
> identify the bottleneck, improve the right thing, and verify that
> correctness has not changed.**

``` text
Correct implementation
      ↓
Representative benchmark
      ↓
Profiler
      ↓
Bottleneck
      ↓
Algorithm / vectorization / memory / parallelism / Numba
      ↓
Tests
      ↓
Rebenchmark
      ↓
Documented improvement
```
