# 7.1 Debugging, Testing & Defensive Programming

## Why this week matters

By Week 13, you have written code that:

-   reshapes and merges data;
-   reads external sources;
-   generates figures;
-   simulates random variables;
-   implements bootstrap and permutation procedures;
-   performs root finding and optimization.

The question is no longer just:

> **Does the code run?**

The more important question is:

> **How do we know the code is correct, robust, and likely to fail
> clearly when its assumptions are violated?**

This week focuses on:

-   systematic debugging;
-   reading tracebacks;
-   assertions and validation;
-   exceptions;
-   defensive programming;
-   unit testing;
-   `pytest`;
-   testing numerical/statistical code;
-   approximate equality;
-   edge cases;
-   regression tests;
-   test organization.

------------------------------------------------------------------------

# 1. Bugs are not all the same

A program can fail in several ways.

## Syntax error

Python cannot parse the code.

``` python
if x > 3
    print(x)
```

## Runtime error

The code parses but fails during execution.

``` python
1 / 0
```

## Logic error

The code runs successfully but produces the wrong answer.

``` python
def mean(values):
    return sum(values) / (
        len(values) - 1
    )
```

Logic errors are often the most dangerous because no exception is
raised.

------------------------------------------------------------------------

# 2. Statistical bugs can be silent

Suppose:

``` python
def standardize(x):
    return (
        x - x.mean()
    ) / x.var()
```

This code runs.

But if the intention is a z-score, the denominator should be the
standard deviation, not variance.

The software executes successfully while the statistical method is
wrong.

Therefore:

> **Successful execution is not evidence of statistical correctness.**

------------------------------------------------------------------------

# 3. Debugging is hypothesis testing

A useful debugging workflow resembles scientific reasoning.

``` text
Observe failure
      ↓
Form hypothesis
      ↓
Collect evidence
      ↓
Test hypothesis
      ↓
Narrow problem
      ↓
Fix
      ↓
Verify
```

Avoid random code changes.

Change one thing at a time and test the result.

------------------------------------------------------------------------

# 4. Read the traceback

When Python raises an exception, it produces a traceback.

The traceback tells you:

-   which exception occurred;
-   the error message;
-   which file/function was executing;
-   the sequence of calls leading to the error;
-   the line where the failure surfaced.

A good habit is:

> **Read the final line first, then work upward through the call
> stack.**

------------------------------------------------------------------------

# 5. Minimal reproducible examples

When debugging, reduce the problem.

Instead of debugging:

``` text
500-line notebook
+
3 data files
+
API calls
+
plots
```

create the smallest example that still fails.

This makes the causal structure clearer.

A minimal example is useful for:

-   your own debugging;
-   asking for help;
-   writing tests;
-   reporting bugs.

------------------------------------------------------------------------

# 6. Inspect values and shapes

Many statistical bugs are shape or type bugs.

Useful inspection:

``` python
type(x)
```

``` python
x.shape
```

``` python
x.dtype
```

``` python
len(x)
```

``` python
df.columns
```

``` python
df.isna().sum()
```

Debugging often begins with:

> **What object is this actually?**

------------------------------------------------------------------------

# 7. Use deliberate diagnostics

Temporary diagnostics can help:

``` python
print(
    "beta:",
    beta
)
```

``` python
print(
    "gradient shape:",
    gradient.shape
)
```

But do not let permanent production code become dominated by debugging
prints.

For larger projects, structured logging can be preferable.

------------------------------------------------------------------------

# 8. Assertions

An assertion states that something must be true.

``` python
assert len(x) > 0
```

``` python
assert X.shape[0] == len(y)
```

``` python
assert np.isfinite(
    X
).all()
```

Assertions are useful for checking assumptions during development and
testing.

------------------------------------------------------------------------

# 9. Assertions communicate invariants

An **invariant** is a condition expected to remain true.

Examples:

``` text
IDs are unique
probabilities lie in [0,1]
matrix dimensions match
standard deviations are positive
resampling count is positive
loss is finite
```

Encode them:

``` python
assert df["id"].is_unique
```

``` python
assert (
    0 <= p <= 1
)
```

Assertions convert assumptions into executable checks.

------------------------------------------------------------------------

# 10. Assertions vs. input validation

Assertions are primarily developer checks.

For public-facing or reusable function input validation, explicit
exceptions are often better.

Instead of:

``` python
assert n_boot > 0
```

prefer:

``` python
if n_boot <= 0:
    raise ValueError(
        "n_boot must be positive."
    )
```

Why?

Assertions can be disabled in optimized Python execution.

Explicit validation is part of the function's user-facing behavior.

------------------------------------------------------------------------

# 11. Exceptions

An exception signals an abnormal condition.

Common built-in exceptions include:

``` text
ValueError
TypeError
KeyError
IndexError
FileNotFoundError
ZeroDivisionError
```

Use the most appropriate exception type.

Example:

``` python
if learning_rate <= 0:
    raise ValueError(
        "learning_rate must be positive."
    )
```

------------------------------------------------------------------------

# 12. Raise early

Suppose a function requires:

``` text
n >= 2
```

Check immediately:

``` python
if n < 2:
    raise ValueError(
        "n must be at least 2."
    )
```

Do not wait for a confusing failure 40 lines later.

This principle is often called:

> **Fail fast.**

------------------------------------------------------------------------

# 13. Good error messages

Bad:

``` python
raise ValueError(
    "Bad input."
)
```

Better:

``` python
raise ValueError(
    "n_boot must be a positive integer; "
    f"received {n_boot}."
)
```

Useful error messages answer:

``` text
What was wrong?
What was expected?
What was received?
```

------------------------------------------------------------------------

# 14. `try` and `except`

Sometimes failure is expected and can be handled.

``` python
try:
    value = float(
        user_input
    )
except ValueError:
    print(
        "Input must be numeric."
    )
```

Exception handling lets the program respond deliberately.

------------------------------------------------------------------------

# 15. Catch specific exceptions

Avoid:

``` python
try:
    ...
except:
    ...
```

This catches almost everything, including failures you did not intend to
suppress.

Prefer:

``` python
except FileNotFoundError:
```

or:

``` python
except ValueError:
```

Catch only errors you know how to handle.

------------------------------------------------------------------------

# 16. Do not silently swallow errors

Dangerous:

``` python
try:
    result = compute()
except Exception:
    result = None
```

Now the program continues without preserving the reason for failure.

Silent failures are especially dangerous in statistical pipelines.

If you cannot recover meaningfully, let the exception propagate or raise
a clearer one.

------------------------------------------------------------------------

# 17. `else` and `finally`

A `try` block may include:

``` python
try:
    ...
except ValueError:
    ...
else:
    ...
finally:
    ...
```

`else` runs when no exception occurred.

`finally` runs regardless of success or failure.

This is useful for resource cleanup.

------------------------------------------------------------------------

# 18. Custom exceptions

For a larger package, domain-specific exceptions can improve clarity.

``` python
class ConvergenceError(
    RuntimeError
):
    pass
```

Then:

``` python
if not converged:
    raise ConvergenceError(
        "Optimizer failed to converge."
    )
```

Custom exceptions can make error handling more expressive.

------------------------------------------------------------------------

# 19. Defensive programming

Defensive programming means anticipating incorrect use or unexpected
states.

Examples:

-   validate inputs;
-   check shapes;
-   check ranges;
-   verify required columns;
-   reject invalid parameter values;
-   detect non-finite values;
-   check convergence;
-   avoid ambiguous mutation.

The goal is not to make errors impossible.

The goal is to make errors **visible, local, and interpretable**.

------------------------------------------------------------------------

# 20. Tests

A test asks:

> **For a known input or condition, does the code produce the expected
> behavior?**

Example:

``` python
def square(x):
    return x ** 2
```

Test:

``` python
assert square(3) == 9
```

A test captures an expectation in executable form.

------------------------------------------------------------------------

# 21. Unit tests

A **unit test** checks a relatively small piece of code, usually a
function.

For:

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

a unit test may use a tiny dataset with a known result.

Small deterministic examples are ideal.

------------------------------------------------------------------------

# 22. `pytest`

`pytest` is a widely used Python testing framework.

The current documentation shows the standard pattern:

``` python
def func(x):
    return x + 1


def test_answer():
    assert func(3) == 5
```

Run from the terminal:

``` bash
pytest
```

`pytest` automatically discovers appropriately named test files and
functions.

------------------------------------------------------------------------

# 23. Test-file organization

A typical project:

``` text
project/
├── src/
│   ├── simulation.py
│   └── optimization.py
├── tests/
│   ├── test_simulation.py
│   └── test_optimization.py
└── notebooks/
```

Common naming:

``` text
test_*.py
```

and:

``` text
def test_something():
```

------------------------------------------------------------------------

# 24. Arrange--Act--Assert

A useful structure is:

``` text
Arrange
Act
Assert
```

Example:

``` python
def test_mean():
    # Arrange
    x = np.array([
        1.0,
        2.0,
        3.0
    ])

    # Act
    result = mean(x)

    # Assert
    assert result == 2.0
```

This makes test intent easy to read.

------------------------------------------------------------------------

# 25. Test normal cases

Example:

``` python
def test_difference_in_means():
    x = np.array([
        4.0,
        6.0
    ])

    y = np.array([
        1.0,
        3.0
    ])

    result = (
        difference_in_means(
            x,
            y
        )
    )

    assert result == 3.0
```

Use examples where the expected answer is transparent.

------------------------------------------------------------------------

# 26. Test edge cases

Edge cases are inputs near the boundaries of expected behavior.

Examples:

``` text
empty arrays
one observation
all equal values
missing values
zero variance
negative parameter
extremely large values
```

Many bugs occur at boundaries rather than typical cases.

------------------------------------------------------------------------

# 27. Test invalid inputs

If a function promises to reject bad input, test that behavior.

With `pytest`:

``` python
import pytest
```

``` python
with pytest.raises(
    ValueError
):
    bootstrap_statistic(
        data=x,
        statistic=np.mean,
        n_boot=0,
        rng=rng
    )
```

The exception is part of the function's contract.

------------------------------------------------------------------------

# 28. Floating-point tests

Avoid:

``` python
assert result == 0.1 + 0.2
```

for numerical algorithms.

Floating-point arithmetic is approximate.

Use:

``` python
np.isclose(
    result,
    expected
)
```

or with `pytest`:

``` python
assert result == pytest.approx(
    expected
)
```

The current `pytest` documentation explicitly supports approximate
comparisons for floating-point tests.

------------------------------------------------------------------------

# 29. Test mathematical properties

Sometimes no simple expected output is available.

Test a property instead.

For QR decomposition:

``` python
assert np.allclose(
    Q @ R,
    X
)
```

and:

``` python
assert np.allclose(
    Q.T @ Q,
    np.eye(
        Q.shape[1]
    )
)
```

These test the mathematics rather than hard-coded values.

------------------------------------------------------------------------

# 30. Property tests for statistics

Examples:

Bootstrap distribution:

``` text
number of replicates = n_boot
```

Probability:

``` text
0 ≤ p ≤ 1
```

Standard deviation:

``` text
sd ≥ 0
```

Optimization:

``` text
final loss ≤ initial loss
```

Permutation:

``` text
null distribution has expected length
```

Statistical code often has useful invariants.

------------------------------------------------------------------------

# 31. Deterministic tests for random code

Random simulation is harder to test unless randomness is controlled.

Pass an RNG:

``` python
rng = np.random.default_rng(
    556
)
```

Then test reproducibly.

This is another reason Week 10 used explicit generator objects.

------------------------------------------------------------------------

# 32. Avoid fragile random tests

Bad:

``` python
x = rng.normal(
    size=100
)

assert x.mean() == 0
```

This is statistically wrong as a test.

Better:

-   test deterministic behavior using a fixed RNG;
-   test shapes;
-   test support constraints;
-   test exact algorithms where possible;
-   use tolerances when testing stochastic properties.

------------------------------------------------------------------------

# 33. Test resampling code

For a bootstrap function, test:

-   correct output length;
-   replacement behavior;
-   deterministic output under a fixed seed;
-   rejection of invalid `n_boot`;
-   expected statistic for a trivial dataset.

For permutation code, test:

-   correct output length;
-   preservation of total observations;
-   deterministic behavior under fixed RNG;
-   p-value bounds;
-   known exact toy examples.

------------------------------------------------------------------------

# 34. Test optimization code

For bisection:

``` text
known root
invalid bracket
maximum iterations
tolerance
```

For Newton:

``` text
known root
near-zero derivative
starting-value behavior
```

For gradient descent:

``` text
loss decreases
known quadratic minimum
invalid learning rate
```

Use simple mathematical examples before real statistical models.

------------------------------------------------------------------------

# 35. Regression tests

A regression test prevents a previously fixed bug from returning.

Workflow:

``` text
discover bug
      ↓
write failing test reproducing bug
      ↓
fix code
      ↓
test passes
      ↓
keep test forever
```

This converts past failures into future protection.

------------------------------------------------------------------------

# 36. Test-driven bug fixing

When a bug appears:

1.  reproduce it;
2.  write a test that fails;
3.  fix the code;
4.  confirm the new test passes;
5.  run the whole test suite.

This prevents the fix from breaking something else.

------------------------------------------------------------------------

# 37. Test independence

Tests should not rely on execution order.

Bad:

``` text
test A creates global data
test B assumes it exists
```

Each test should arrange its own required state or use explicit
fixtures.

Independent tests are easier to debug and parallelize.

------------------------------------------------------------------------

# 38. Fixtures

`pytest` fixtures provide reusable test setup.

Example:

``` python
import pytest


@pytest.fixture
def sample_data():
    return np.array([
        1.0,
        2.0,
        3.0
    ])
```

Then:

``` python
def test_mean(
    sample_data
):
    assert (
        sample_data.mean()
        == 2.0
    )
```

Fixtures reduce repeated setup.

------------------------------------------------------------------------

# 39. Parametrized tests

Suppose:

``` python
def square(x):
    return x ** 2
```

Instead of writing many similar tests:

``` python
@pytest.mark.parametrize(
    "x, expected",
    [
        (0, 0),
        (1, 1),
        (2, 4),
        (-3, 9)
    ]
)
def test_square(
    x,
    expected
):
    assert (
        square(x)
        == expected
    )
```

Parametrization makes systematic case coverage concise.

------------------------------------------------------------------------

# 40. Tests should be readable

A test suite is documentation.

A reader should understand:

-   intended inputs;
-   expected outputs;
-   valid ranges;
-   expected errors;
-   numerical tolerance;
-   statistical assumptions.

Good tests explain what the code promises.

------------------------------------------------------------------------

# 41. What not to test

Avoid tests that simply repeat library behavior.

For example, if your function is:

``` python
def mean(x):
    return np.mean(x)
```

a detailed test suite may add little value unless the wrapper enforces a
meaningful contract.

Test your logic and assumptions, not every line indiscriminately.

------------------------------------------------------------------------

# 42. Coverage is not correctness

High test coverage means much code was executed during tests.

It does **not** mean:

``` text
the tests are good
```

or:

``` text
the statistical method is correct
```

Coverage is useful as a diagnostic, not a guarantee.

------------------------------------------------------------------------

# 43. Debugging notebooks vs. modules

Notebooks are useful for exploration.

But reusable code should increasingly live in:

``` text
src/
```

and tests in:

``` text
tests/
```

This makes errors easier to isolate.

A good workflow is:

``` text
notebook discovers idea
      ↓
function moves to module
      ↓
tests define contract
      ↓
notebook imports tested function
```

------------------------------------------------------------------------

# 44. Continuous verification mindset

Every new function should trigger questions:

``` text
What should happen for a normal case?
What should happen at the boundary?
What input should fail?
What mathematical property should hold?
What previously observed bug should never return?
```

Testing becomes part of development rather than something added
afterward.

------------------------------------------------------------------------

# 45. Recommended reading

## pytest --- Getting Started

The official guide introduces installation, first tests, discovery,
assertions, exception tests, and approximate comparisons.

https://docs.pytest.org/en/stable/getting-started.html

## pytest --- Full Documentation

https://docs.pytest.org/en/stable/

## Python --- Errors and Exceptions

The official Python tutorial covers syntax errors, exceptions,
`try`/`except`, raising exceptions, and cleanup behavior.

https://docs.python.org/3/tutorial/errors.html

## Real Python --- Raising and Handling Exceptions

A practical guide to exceptions, assertions, custom exceptions,
`try`/`except`, `else`, and `finally`.

https://realpython.com/courses/raising-handling-exceptions/

------------------------------------------------------------------------

# 46. YouTube recommendations

## 1. Tech With Tim --- "Please Learn How To Write Tests in Python... • Pytest Tutorial"

A recent practical introduction to `pytest`, including assertions,
fixtures, parametrization, mocking, and testing APIs.

**Recommended use:** Watch the first \~20 minutes alongside the
introductory `pytest`, fixture, and parametrization material.

[Watch on YouTube](https://www.youtube.com/watch?v=EgpLj86ZHFQ)

------------------------------------------------------------------------

## 2. freeCodeCamp.org --- "Pytest Tutorial --- How to Test Python Code"

A longer structured course covering first tests, fixtures,
parametrization, mocking, and broader test organization.

**Recommended use:** Optional deeper reference for students who want
more practice after the tutorial.

[Watch on YouTube](https://www.youtube.com/watch?v=cHYq1MRoyI0)

------------------------------------------------------------------------

## 3. Dave Gray --- "Python Exception Handling Tutorial for Beginners"

A concise treatment of built-in exceptions, `try`/`except`, `else`,
`finally`, and custom exceptions.

**Recommended use:** Watch before the exception-handling exercises.

[Watch on YouTube](https://www.youtube.com/watch?v=PHzm_Iox1mE)

------------------------------------------------------------------------

# 47. Week 13 takeaway

The central lesson is:

> **Reliable statistical software does not merely produce answers; it
> makes assumptions explicit, detects violations, and provides evidence
> that important behaviors remain correct.**

``` text
Code
 ↓
Assumptions
 ↓
Validation
 ↓
Tests
 ↓
Failure cases
 ↓
Debug
 ↓
Fix
 ↓
Regression test
 ↓
More reliable workflow
```

Next week we move into **performance, profiling, and acceleration**,
where we will measure bottlenecks before deciding how to make
statistical code faster.
