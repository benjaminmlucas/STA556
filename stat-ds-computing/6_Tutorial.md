# 4.1 Activity

## Modular Programming & Functions

**Estimated time:** 2–3 hours  
**Tools:** Python, pandas, Jupyter/VS Code

## Learning objectives

By the end of this tutorial, you should be able to:

- define reusable Python functions;
- distinguish parameters from arguments;
- return values rather than relying on printing;
- use defaults, keyword arguments, type hints, and docstrings;
- recognize pure functions and side effects;
- understand local and global scope;
- reduce dependence on global variables;
- compose small functions into workflows;
- use `map()`, `filter()`, and `reduce()`;
- recognize when comprehensions are more readable;
- apply functions to pandas data;
- move reusable functions into modules.

---

# Part 0 — Set up

Create:

```text
notebooks/week07_functions.ipynb
```

and:

```text
src/week07_utils.py
```

Import:

```python
import pandas as pd
import numpy as np
```

---

# Part 1 — Repeated code

```python
scores_a = [81, 88, 92, 79]
scores_b = [74, 85, 90, 95]
```

Calculate:

```python
mean_a = sum(scores_a) / len(scores_a)
mean_b = sum(scores_b) / len(scores_b)
```

### Question

What is duplicated here?

What happens if this calculation occurs in 20 places?

---

# Part 2 — Write a function

```python
def mean_score(values):
    return sum(values) / len(values)
```

Run:

```python
mean_score(scores_a)
mean_score(scores_b)
```

What has been abstracted away?

---

# Part 3 — Parameters and arguments

For:

```python
def mean_score(values):
    return sum(values) / len(values)
```

identify:

```text
function name:
parameter:
argument in mean_score(scores_a):
```

---

# Part 4 — Return vs. print

Compare:

```python
def print_mean(values):
    print(
        sum(values) / len(values)
    )
```

with:

```python
def return_mean(values):
    return (
        sum(values) / len(values)
    )
```

Run:

```python
x = print_mean(scores_a)
y = return_mean(scores_a)
```

Inspect:

```python
x
y
```

Why is the returned result more reusable?

---

# Part 5 — Multiple outputs

```python
def summarize(values):
    return (
        min(values),
        max(values),
        sum(values) / len(values)
    )
```

Then:

```python
minimum, maximum, mean = (
    summarize(scores_a)
)
```

### Challenge

Also return the number of observations.

---

# Part 6 — Default arguments

```python
def classify_score(
    score,
    threshold=80
):
    if score >= threshold:
        return "high"

    return "low"
```

Try:

```python
classify_score(85)
classify_score(85, threshold=90)
```

What behavior does the default encode?

---

# Part 7 — Keyword arguments

```python
def simulate_scores(
    n,
    mean,
    sd,
    seed
):
    rng = np.random.default_rng(seed)

    return rng.normal(
        mean,
        sd,
        n
    )
```

Compare:

```python
simulate_scores(
    100,
    80,
    10,
    556
)
```

with:

```python
simulate_scores(
    n=100,
    mean=80,
    sd=10,
    seed=556
)
```

Which is easier to interpret?

---

# Part 8 — Type hints

```python
def classify_score(
    score: float,
    threshold: float = 80
) -> str:
    if score >= threshold:
        return "high"

    return "low"
```

Inspect:

```python
classify_score.__annotations__
```

What information do the annotations provide?

---

# Part 9 — Docstrings

```python
def classify_score(
    score: float,
    threshold: float = 80
) -> str:
    """Classify a score using a threshold."""
    if score >= threshold:
        return "high"

    return "low"
```

Inspect:

```python
help(classify_score)
```

### Challenge

Expand the docstring to describe:

- `score`;
- `threshold`;
- return value.

---

# Part 10 — Pure function

```python
def square(x):
    return x ** 2
```

Run:

```python
square(4)
square(4)
square(4)
```

Does anything outside the function change?

---

# Part 11 — Side effects

```python
results = []
```

```python
def record_result(x):
    results.append(x)
```

Run:

```python
record_result(10)
record_result(20)
results
```

What external state changed?

---

# Part 12 — Mutation of inputs

```python
def append_score(
    scores,
    score
):
    scores.append(score)
```

Then:

```python
values = [80, 90]

append_score(
    values,
    100
)

values
```

Why could this surprise a caller?

---

# Part 13 — Non-mutating alternative

```python
def append_score(
    scores,
    score
):
    return scores + [score]
```

Then:

```python
values = [80, 90]

new_values = append_score(
    values,
    100
)
```

Inspect:

```python
values
new_values
```

Which is easier to reason about?

---

# Part 14 — Local scope

```python
def calculate():
    local_value = 42
    return local_value
```

Run:

```python
calculate()
```

Then try:

```python
local_value
```

Why is the name unavailable outside the function?

---

# Part 15 — Global scope

```python
threshold = 80
```

```python
def classify(score):
    if score >= threshold:
        return "high"

    return "low"
```

Run:

```python
classify(85)
```

Change:

```python
threshold = 90
```

Run again:

```python
classify(85)
```

Why did the same function call change behavior?

---

# Part 16 — Remove hidden global dependence

Rewrite:

```python
def classify(
    score,
    threshold=80
):
    if score >= threshold:
        return "high"

    return "low"
```

Now:

```python
classify(85)
classify(85, threshold=90)
```

Why is this interface clearer?

---

# Part 17 — LEGB exploration

```python
x = "global"
```

```python
def outer():
    x = "enclosing"

    def inner():
        x = "local"
        return x

    return inner()
```

Run:

```python
outer()
```

Remove the local `x` from `inner()` and repeat.

Then remove the enclosing `x`.

Complete:

```text
L:
E:
G:
B:
```

Explain the lookup order.

---

# Part 18 — Refactor a large function

Start:

```python
def analyze(data):
    clean = data.dropna(
        subset=["age", "score"]
    )

    clean = clean.loc[
        clean["age"] >= 18
    ]

    mean = clean["score"].mean()

    return mean
```

Split it into:

```python
def remove_missing(data):
    ...
```

```python
def select_adults(data):
    ...
```

```python
def mean_score(data):
    ...
```

---

# Part 19 — Compose the functions

Use:

```python
clean = remove_missing(df)
adults = select_adults(clean)
result = mean_score(adults)
```

Why might this be easier to test than one large function?

---

# Part 20 — Create a DataFrame

```python
df = pd.DataFrame({
    "id": [1, 2, 3, 4, 5, 6],
    "age": [24, 31, 17, 42, 29, 35],
    "group": ["A", "B", "A", "B", "A", "B"],
    "score": [81, 94, 88, np.nan, 91, 85]
})
```

Use your functions from the previous parts.

---

# Part 21 — Functions are objects

```python
def square(x):
    return x ** 2
```

Then:

```python
f = square
f(5)
type(square)
```

What does it mean to say functions are first-class objects?

---

# Part 22 — Higher-order function

```python
def apply_twice(
    function,
    value
):
    return function(
        function(value)
    )
```

Run:

```python
apply_twice(
    square,
    2
)
```

### Challenge

```python
def increment(x):
    return x + 1
```

Predict:

```python
apply_twice(
    increment,
    10
)
```

before running it.

---

# Part 23 — `map()`

```python
values = [1, 2, 3, 4, 5]
```

```python
mapped = map(
    square,
    values
)
```

Inspect:

```python
mapped
list(mapped)
```

Why does `map()` not immediately return a list?

---

# Part 24 — `map()` vs. comprehension

Compare:

```python
list(
    map(
        square,
        values
    )
)
```

with:

```python
[
    square(x)
    for x in values
]
```

Which is clearer?

---

# Part 25 — `filter()`

```python
def is_even(x):
    return x % 2 == 0
```

```python
filtered = filter(
    is_even,
    values
)

list(filtered)
```

Now write the equivalent comprehension.

---

# Part 26 — Statistical predicate

```python
def is_high_score(
    score,
    threshold=85
):
    return score >= threshold
```

Use:

```python
scores = [72, 81, 91, 68, 88, 95]
```

Then:

```python
list(
    filter(
        is_high_score,
        scores
    )
)
```

Compare with a comprehension.

---

# Part 27 — `reduce()`

```python
from functools import reduce
```

```python
def add(x, y):
    return x + y
```

Run:

```python
reduce(
    add,
    [1, 2, 3, 4]
)
```

Write the sequence of reductions manually.

---

# Part 28 — Reduction and statistics

```python
scores = [81, 94, 88, 91]
```

Calculate:

```python
total = reduce(
    add,
    scores
)

mean = total / len(scores)
```

Why is built-in `sum()` preferable here?

The goal is to understand reduction rather than replace clearer built-ins.

---

# Part 29 — Lambda expressions

Compare:

```python
def square(x):
    return x ** 2
```

with:

```python
lambda x: x ** 2
```

Use:

```python
list(
    map(
        lambda x: x ** 2,
        values
    )
)
```

### Challenge

Use a lambda with `filter()` to retain values above 3.

---

# Part 30 — When not to use lambda

Would this be clear?

```python
lambda x: (
    complicated_transformation(x)
    if complicated_condition(x)
    else another_transformation(x)
)
```

> Use lambdas for short, local behavior—not to hide complicated logic.

---

# Part 31 — Apply a function to pandas

```python
def categorize_score(score):
    if pd.isna(score):
        return "missing"

    if score >= 90:
        return "high"

    if score >= 80:
        return "medium"

    return "low"
```

Apply:

```python
df["performance"] = (
    df["score"]
    .map(categorize_score)
)
```

Inspect `df`.

---

# Part 32 — Reusable DataFrame function

```python
def select_group(
    data: pd.DataFrame,
    group: str
) -> pd.DataFrame:
    return data.loc[
        data["group"] == group
    ].copy()
```

Test:

```python
select_group(df, "A")
select_group(df, "B")
```

---

# Part 33 — Add a defensive check

```python
def select_group(
    data: pd.DataFrame,
    group: str
) -> pd.DataFrame:
    if "group" not in data.columns:
        raise ValueError(
            "data must contain a group column"
        )

    return data.loc[
        data["group"] == group
    ].copy()
```

Test on a DataFrame without `group`.

Why is an informative error useful?

---

# Part 34 — Build an analysis pipeline

```python
def remove_missing_scores(data):
    return data.dropna(
        subset=["score"]
    ).copy()
```

```python
def select_adults(data):
    return data.loc[
        data["age"] >= 18
    ].copy()
```

```python
def summarize_scores(data):
    return {
        "n": len(data),
        "mean": data["score"].mean(),
        "sd": data["score"].std()
    }
```

Use:

```python
clean = remove_missing_scores(df)
adults = select_adults(clean)
summary = summarize_scores(adults)
```

---

# Part 35 — One-expression composition

Try:

```python
summary = summarize_scores(
    select_adults(
        remove_missing_scores(df)
    )
)
```

Is this clearer or less clear than named intermediate objects?

Readable code matters more than compact code.

---

# Part 36 — Move functions into a module

Open:

```text
src/week07_utils.py
```

Move these functions into it:

```text
categorize_score
select_group
remove_missing_scores
select_adults
summarize_scores
```

---

# Part 37 — Import your module

From the notebook:

```python
from src.week07_utils import (
    categorize_score,
    select_group,
    remove_missing_scores,
    select_adults,
    summarize_scores
)
```

Use one function.

How has the notebook changed now that reusable logic lives elsewhere?

---

# Part 38 — Inspect the module boundary

Your project now resembles:

```text
project/
├── notebooks/
│   └── week07_functions.ipynb
├── src/
│   └── week07_utils.py
├── data/
├── figures/
└── tests/
```

Which code belongs in the notebook?

Which belongs in `src/`?

---

# Part 39 — Mini refactoring challenge

Start with:

```python
df = pd.read_csv("data/study.csv")

df = df.dropna(
    subset=["score"]
)

df = df.loc[
    df["age"] >= 18
]

df["performance"] = (
    df["score"]
    .map(categorize_score)
)

summary = {
    "n": len(df),
    "mean": df["score"].mean(),
    "sd": df["score"].std()
}
```

Refactor into a workflow resembling:

```python
raw = load_data(...)
clean = clean_data(raw)
analysis = prepare_analysis(clean)
summary = summarize_scores(analysis)
```

---

# Part 40 — Function design critique

For each function, ask:

### Interface

- Are inputs explicit?
- Is the return value clear?

### Purity

- Does it unexpectedly mutate data?
- Does it rely on globals?

### Scope

- Are internal variables local?

### Responsibility

- Does it do one coherent job?

### Documentation

- Does it need a docstring?
- Would type hints improve clarity?

Revise at least two functions.

---

# Part 41 — Git checkpoint

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
git commit -m "Complete Week 7 modular programming exercises"
```

Push:

```bash
git push
```

---

# Part 42 — Final reflection

### 1. Functions

Why are functions useful in statistical workflows?

### 2. Return values

Why is returning generally more reusable than printing?

### 3. Purity

What makes a function pure?

### 4. Side effects

Give two examples.

### 5. Scope

What is the difference between local and global scope?

### 6. Global state

Why can global variables make functions harder to test?

### 7. Higher-order functions

What does it mean for a function to accept another function?

### 8. Functional patterns

Explain `map`, `filter`, and `reduce` conceptually.

### 9. Python style

Why might a comprehension be preferable to `map()` or `filter()`?

### 10. Modularity

Why move functions from notebooks into modules?

---

# Completion checklist

- [ ] Created Week 7 notebook
- [ ] Created a Week 7 Python module
- [ ] Refactored duplicated code into functions
- [ ] Distinguished parameters and arguments
- [ ] Used return values
- [ ] Returned multiple values
- [ ] Used default arguments
- [ ] Used keyword arguments
- [ ] Added type hints
- [ ] Added docstrings
- [ ] Identified a pure function
- [ ] Identified side effects
- [ ] Compared mutating and non-mutating functions
- [ ] Explored local and global scope
- [ ] Demonstrated the LEGB rule
- [ ] Removed hidden global dependencies
- [ ] Split one large function into smaller ones
- [ ] Composed functions
- [ ] Treated a function as an object
- [ ] Wrote a higher-order function
- [ ] Used `map()`
- [ ] Used `filter()`
- [ ] Used `reduce()`
- [ ] Compared functional tools with comprehensions
- [ ] Used lambda expressions
- [ ] Applied a function to a pandas Series
- [ ] Wrote a reusable DataFrame function
- [ ] Added a defensive validation check
- [ ] Built a modular analysis pipeline
- [ ] Imported functions from `src/`
- [ ] Refactored an analysis into reusable components
- [ ] Committed Week 7 work to Git
- [ ] Pushed work to GitHub

---

# What you should now understand

```text
Repeated code
      ↓
Functions
      ↓
Explicit inputs + outputs
      ↓
Pure transformations
      ↓
Small reusable components
      ↓
Composition
      ↓
Modules
      ↓
Maintainable statistical software
```

Next week we will use these modular components inside **reproducible reports and literate programming workflows**.
