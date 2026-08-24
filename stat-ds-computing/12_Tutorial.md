# 7.1 Activity

## Debugging, Testing & Defensive Programming

**Tools:** Python, NumPy, pandas, pytest, Jupyter/VS Code, terminal

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

-   distinguish syntax, runtime, and logic errors;
-   read Python tracebacks;
-   reduce bugs to minimal examples;
-   validate assumptions using assertions and exceptions;
-   write defensive statistical functions;
-   create `pytest` unit tests;
-   test expected exceptions;
-   use approximate numerical comparisons;
-   test deterministic and stochastic code;
-   write fixtures and parametrized tests;
-   add regression tests for discovered bugs;
-   organize tests alongside reusable modules.

------------------------------------------------------------------------

# Part 0 --- Set up the Week 13 project

Create:

``` text
src/week13_utils.py
tests/test_week13_utils.py
notebooks/week13_testing.ipynb
```

Check:

``` bash
pytest --version
```

`pytest` is included in the STA 556 Codespace. If `pytest --version` fails in the official course Codespace, report the environment issue rather than installing packages ad hoc.

Run:

``` bash
pytest
```

You should initially see that no tests were collected.

------------------------------------------------------------------------

# Part 1 --- Three kinds of failure

Consider:

### A

``` python
if x > 2
    print(x)
```

### B

``` python
x = 1 / 0
```

### C

``` python
def variance(x):
    return np.std(
        x
    )
```

when the function is supposed to return variance.

Classify each as:

``` text
syntax error
runtime error
logic error
```

Which is hardest to detect automatically?

------------------------------------------------------------------------

# Part 2 --- Read a traceback

Run:

``` python
def divide(
    x,
    y
):
    return x / y


def calculate():
    return divide(
        10,
        0
    )


calculate()
```

Read the traceback from the bottom upward.

Identify:

``` text
exception type
error message
line where failure occurred
function-call chain
```

------------------------------------------------------------------------

# Part 3 --- Create a minimal example

Suppose a large analysis fails during:

``` python
X @ beta
```

Reduce it to:

``` python
X = np.ones(
    (
        5,
        3
    )
)

beta = np.ones(
    4
)

X @ beta
```

### Question

Why is this small example easier to debug?

Fix the shape problem.

------------------------------------------------------------------------

# Part 4 --- Debug with object inspection

Create:

``` python
x = np.array([
    1,
    2,
    3
])

X = x.reshape(
    -1,
    1
)
```

Inspect:

``` python
type(x)
x.shape
X.shape
x.dtype
```

### Reflection

How might shape inspection have prevented a Week 9 broadcasting bug?

------------------------------------------------------------------------

# Part 5 --- Assertions

Create:

``` python
def standardize(
    x
):
    x = np.asarray(
        x,
        dtype=float
    )

    assert x.ndim == 1
    assert len(x) > 0

    return (
        x - x.mean()
    ) / x.std()
```

Test with:

``` python
standardize([
    1,
    2,
    3
])
```

Then try:

``` python
standardize(
    np.ones(
        (
            2,
            2
        )
    )
)
```

------------------------------------------------------------------------

# Part 6 --- Identify a missing invariant

Try:

``` python
standardize([
    5,
    5,
    5
])
```

What happens?

Add a check for zero standard deviation.

### Question

What invariant was missing?

------------------------------------------------------------------------

# Part 7 --- Replace developer assertions with validation

Rewrite:

``` python
def standardize(
    x
):
    x = np.asarray(
        x,
        dtype=float
    )

    if x.ndim != 1:
        raise ValueError(
            "x must be one-dimensional."
        )

    if len(x) == 0:
        raise ValueError(
            "x must not be empty."
        )

    sd = x.std()

    if sd == 0:
        raise ValueError(
            "x must have positive variability."
        )

    return (
        x - x.mean()
    ) / sd
```

Why is this a better public function interface?

------------------------------------------------------------------------

# Part 8 --- Improve error messages

Modify each validation message to include the received value/shape where
helpful.

For example:

``` python
raise ValueError(
    "x must be one-dimensional; "
    f"received shape {x.shape}."
)
```

### Principle

Good errors make debugging faster.

------------------------------------------------------------------------

# Part 9 --- Exception handling

Create:

``` python
def read_numeric(
    text
):
    try:
        value = float(
            text
        )
    except ValueError:
        return None
    else:
        return value
```

Test:

``` python
read_numeric(
    "3.14"
)
```

and:

``` python
read_numeric(
    "hello"
)
```

------------------------------------------------------------------------

# Part 10 --- Catch specific exceptions

Compare:

``` python
except:
```

with:

``` python
except ValueError:
```

### Question

Why is a bare `except` dangerous?

------------------------------------------------------------------------

# Part 11 --- Do not hide failures

Consider:

``` python
def safe_analysis(
    data
):
    try:
        return run_analysis(
            data
        )
    except Exception:
        return None
```

List three reasons this could be dangerous in a research pipeline.

Rewrite it to handle only an error you genuinely know how to recover
from.

------------------------------------------------------------------------

# Part 12 --- Create a convergence exception

In `src/week13_utils.py`:

``` python
class ConvergenceError(
    RuntimeError
):
    pass
```

Then create:

``` python
def check_convergence(
    converged
):
    if not converged:
        raise ConvergenceError(
            "Numerical algorithm "
            "failed to converge."
        )
```

Test manually.

------------------------------------------------------------------------

# Part 13 --- First pytest test

Add to `src/week13_utils.py`:

``` python
def square(x):
    return x ** 2
```

In:

``` text
tests/test_week13_utils.py
```

write:

``` python
from src.week13_utils import (
    square
)


def test_square():
    assert square(3) == 9
```

Run:

``` bash
pytest
```

------------------------------------------------------------------------

# Part 14 --- Make a test fail deliberately

Change:

``` python
assert square(3) == 10
```

Run:

``` bash
pytest
```

Read pytest's failure output.

Restore the correct expected value.

### Question

How does the test output differ from a generic exception traceback?

------------------------------------------------------------------------

# Part 15 --- Arrange--Act--Assert

Rewrite:

``` python
def test_square():
    # Arrange
    x = 4

    # Act
    result = square(
        x
    )

    # Assert
    assert result == 16
```

Why might this structure help when tests become more complicated?

------------------------------------------------------------------------

# Part 16 --- Test `standardize`

Move `standardize()` into:

``` text
src/week13_utils.py
```

Write:

``` python
def test_standardize_mean():
    x = np.array([
        1.0,
        2.0,
        3.0
    ])

    z = standardize(
        x
    )

    assert np.isclose(
        z.mean(),
        0.0
    )
```

Add a second test for standard deviation.

------------------------------------------------------------------------

# Part 17 --- Test approximate equality with pytest

Import:

``` python
import pytest
```

Then:

``` python
def test_standardize_sd():
    x = np.array([
        1.0,
        2.0,
        3.0
    ])

    z = standardize(
        x
    )

    assert z.std() == pytest.approx(
        1.0
    )
```

### Question

Why not use exact floating-point equality?

------------------------------------------------------------------------

# Part 18 --- Test invalid input

Write:

``` python
def test_standardize_constant():
    with pytest.raises(
        ValueError
    ):
        standardize([
            5,
            5,
            5
        ])
```

Run:

``` bash
pytest
```

The error behavior is now part of the tested contract.

------------------------------------------------------------------------

# Part 19 --- Test the error message

Use:

``` python
with pytest.raises(
    ValueError,
    match="positive variability"
):
    standardize([
        5,
        5,
        5
    ])
```

Why might checking the message be useful?

When could it make a test too brittle?

------------------------------------------------------------------------

# Part 20 --- Parametrized tests

Write:

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
def test_square_many(
    x,
    expected
):
    assert (
        square(x)
        == expected
    )
```

Run the test suite.

### Reflection

What repetition did parametrization remove?

------------------------------------------------------------------------

# Part 21 --- Create a fixture

Add:

``` python
@pytest.fixture
def sample_scores():
    return np.array([
        70.0,
        80.0,
        90.0
    ])
```

Use:

``` python
def test_score_mean(
    sample_scores
):
    assert (
        sample_scores.mean()
        == 80.0
    )
```

What role does the fixture play?

------------------------------------------------------------------------

# Part 22 --- Test a Week 11 statistic

Add:

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

Test with:

``` python
x = np.array([
    4.0,
    6.0
])

y = np.array([
    1.0,
    3.0
])
```

Expected difference:

``` text
3
```

------------------------------------------------------------------------

# Part 23 --- Add defensive validation

Modify:

``` python
def difference_in_means(
    x,
    y
):
```

so that empty inputs raise `ValueError`.

Write tests for both:

``` text
empty x
empty y
```

------------------------------------------------------------------------

# Part 24 --- Test random code deterministically

Write:

``` python
def simulate_normal(
    n,
    rng
):
    if n <= 0:
        raise ValueError(
            "n must be positive."
        )

    return rng.normal(
        size=n
    )
```

Test:

``` python
def test_simulate_normal_shape():
    rng = np.random.default_rng(
        556
    )

    result = simulate_normal(
        10,
        rng
    )

    assert result.shape == (
        10,
    )
```

------------------------------------------------------------------------

# Part 25 --- Test reproducibility

Write:

``` python
def test_simulate_normal_reproducible():
    rng1 = np.random.default_rng(
        556
    )

    rng2 = np.random.default_rng(
        556
    )

    x1 = simulate_normal(
        5,
        rng1
    )

    x2 = simulate_normal(
        5,
        rng2
    )

    assert np.array_equal(
        x1,
        x2
    )
```

Why is this stronger than checking that a random sample mean is exactly
zero?

------------------------------------------------------------------------

# Part 26 --- Test probability bounds

Suppose a function returns a permutation p-value.

Write a property test:

``` python
assert (
    0 <= p_value <= 1
)
```

### Challenge

List three other useful statistical invariants from Weeks 10--12.

------------------------------------------------------------------------

# Part 27 --- Test bisection from Week 12

Move your `bisection()` function into `src/`.

Write a test using:

``` python
def f(x):
    return x ** 2 - 2
```

Expected:

``` python
np.sqrt(2)
```

Use:

``` python
pytest.approx(
    np.sqrt(2),
    abs=1e-7
)
```

------------------------------------------------------------------------

# Part 28 --- Test invalid bisection brackets

Write:

``` python
def test_bisection_invalid_bracket():
    with pytest.raises(
        ValueError
    ):
        bisection(
            f,
            2,
            3
        )
```

This protects the fail-fast behavior you added in Week 12.

------------------------------------------------------------------------

# Part 29 --- Test optimization properties

For the simple loss:

``` python
def objective(x):
    return (
        x - 3
    ) ** 2
```

Test that gradient descent finishes near:

``` text
3
```

Also test:

``` text
final objective <= initial objective
```

What mathematical property is this checking?

------------------------------------------------------------------------

# Part 30 --- Deliberately introduce a bug

Change the gradient from:

``` python
2 * (x - 3)
```

to:

``` python
2 * (x + 3)
```

Run the test suite.

Which test fails?

Restore the correct gradient.

### Reflection

How did the test protect against a logic error that Python itself would
never flag?

------------------------------------------------------------------------

# Part 31 --- Regression test workflow

Introduce a realistic bug:

``` python
standardize(
    [5, 5, 5]
)
```

Suppose this once returned NaNs instead of failing.

Document the workflow:

``` text
bug discovered
      ↓
test written
      ↓
test fails
      ↓
validation added
      ↓
test passes
```

Keep the test.

------------------------------------------------------------------------

# Part 32 --- Test a DataFrame function

Create:

``` python
def require_columns(
    data,
    columns
):
    missing = [
        column
        for column in columns
        if column
        not in data.columns
    ]

    if missing:
        raise ValueError(
            "Missing required columns: "
            + ", ".join(
                missing
            )
        )

    return True
```

Write tests for:

-   all columns present;
-   one missing column;
-   several missing columns.

------------------------------------------------------------------------

# Part 33 --- Minimal datasets

For testing, use tiny datasets.

Example:

``` python
df = pd.DataFrame({
    "id": [1, 2],
    "score": [80, 90]
})
```

Why might this be better than loading a 500 MB production dataset in a
unit test?

------------------------------------------------------------------------

# Part 34 --- Test independence

Create two tests.

Make the first modify a global list.

Make the second depend on the modified list.

Run the suite in different orders if possible.

### Reflection

Why is shared mutable state dangerous in tests?

Rewrite so each test creates its own data.

------------------------------------------------------------------------

# Part 35 --- Build a Week 13 test suite

Your final test file should include tests for:

``` text
square
standardize
difference_in_means
simulate_normal
bisection
gradient descent
required DataFrame columns
```

Include at least:

-   one normal case;
-   one edge case;
-   one invalid-input case;
-   one approximate numerical test;
-   one stochastic/reproducibility test.

------------------------------------------------------------------------

# Part 36 --- Run subsets of tests

Run all:

``` bash
pytest
```

Run one file:

``` bash
pytest tests/test_week13_utils.py
```

Run one test by keyword:

``` bash
pytest -k standardize
```

Use verbose output:

``` bash
pytest -v
```

------------------------------------------------------------------------

# Part 37 --- Debug a failing test

Choose one passing test and temporarily break the corresponding
function.

Then:

1.  run only that test;
2.  read the failure;
3.  inspect inputs/intermediate values;
4.  form a hypothesis;
5.  fix one thing;
6.  rerun;
7.  run the full suite.

This is the core development cycle.

------------------------------------------------------------------------

# Part 38 --- Testing audit

For one statistical function, answer:

``` text
What is its contract?
What inputs are valid?
What inputs are invalid?
What normal case can be checked exactly?
What edge case is important?
What mathematical invariant should hold?
Does it involve floating-point tolerance?
Does it involve randomness?
What past bug should become a regression test?
```

------------------------------------------------------------------------

# Part 39 --- Project structure

Your repository should now look approximately like:

``` text
project/
├── src/
│   └── week13_utils.py
├── tests/
│   └── test_week13_utils.py
├── notebooks/
│   └── week13_testing.ipynb
└── ...
```

Explain why this is cleaner than keeping functions and tests inside one
notebook.

------------------------------------------------------------------------

# Part 40 --- Git checkpoint

Run:

``` bash
pytest
```

Only commit after the suite passes.

Then:

``` bash
git status
git add .
git commit -m "Complete Week 13 debugging and testing exercises"
git push
```

------------------------------------------------------------------------

# Part 41 --- Final reflection

Answer in Markdown.

### 1. Bugs

What is the difference between syntax, runtime, and logic errors?

### 2. Debugging

Why is debugging similar to hypothesis testing?

### 3. Tracebacks

What information does a traceback provide?

### 4. Assertions

What is an invariant?

### 5. Exceptions

When should a function raise an exception rather than continue?

### 6. Defensive programming

What does "fail fast" mean?

### 7. Unit testing

What should a unit test verify?

### 8. Floating point

Why use approximate equality?

### 9. Randomness

How can random code be tested reproducibly?

### 10. Regression tests

Why should a fixed bug usually lead to a new permanent test?

------------------------------------------------------------------------

# Completion checklist

-   [ ] Created Week 13 notebook
-   [ ] Created `src/week13_utils.py`
-   [ ] Created `tests/test_week13_utils.py`
-   [ ] Distinguished syntax/runtime/logic errors
-   [ ] Read a traceback
-   [ ] Built a minimal failing example
-   [ ] Inspected shape/type/value diagnostics
-   [ ] Added assertions
-   [ ] Replaced public assertions with explicit validation
-   [ ] Improved error messages
-   [ ] Used `try`/`except`
-   [ ] Caught specific exceptions
-   [ ] Avoided silent exception swallowing
-   [ ] Created a custom convergence exception
-   [ ] Wrote a first `pytest` test
-   [ ] Deliberately observed a failing test
-   [ ] Used Arrange--Act--Assert
-   [ ] Tested `standardize`
-   [ ] Used `pytest.approx`
-   [ ] Tested expected exceptions
-   [ ] Checked an exception message
-   [ ] Wrote parametrized tests
-   [ ] Used a fixture
-   [ ] Tested difference in means
-   [ ] Tested random simulation deterministically
-   [ ] Tested reproducibility
-   [ ] Tested statistical invariants
-   [ ] Tested bisection
-   [ ] Tested invalid brackets
-   [ ] Tested gradient-descent properties
-   [ ] Introduced and caught a logic bug
-   [ ] Created a regression test
-   [ ] Tested DataFrame requirements
-   [ ] Removed shared mutable state between tests
-   [ ] Built a multi-function test suite
-   [ ] Ran all and selected tests
-   [ ] Debugged a failing test systematically
-   [ ] Audited a function's test design
-   [ ] Ran the full suite before committing
-   [ ] Committed Week 13 work to Git
-   [ ] Pushed work to GitHub

------------------------------------------------------------------------

# What you should now understand

``` text
Statistical function
      ↓
Contract
      ↓
Validation
      ↓
Normal + edge + failure cases
      ↓
Automated tests
      ↓
Bug discovered
      ↓
Regression test
      ↓
More reliable statistical software
```

Next week we move into **profiling, bottleneck identification,
vectorization, parallel computation, and Numba**.
