# 4.1 Modular Programming & Functions

## Why this week matters

The first half of STA 556 has focused on data structures, wrangling, external data, and visualization. As projects grow, repeated blocks of code become difficult to maintain.

The central question for Week 7 is:

> **How do we turn repeated computational steps into reusable, testable, readable components?**

The approved course schedule identifies Week 7 as **Modular Programming & Functions**, including:

- writing pure functions;
- variable scope;
- functional programming concepts;
- `map`, `filter`, and `reduce`.

The goal is not simply to learn Python's `def` syntax. It is to begin thinking like a software engineer working on statistical code.

---

# 1. Why functions?

Suppose we repeatedly calculate a standardized score:

```python
mean = values.mean()
sd = values.std()
z = (values - mean) / sd
```

If this operation occurs in several places, copying and pasting creates problems:

- changes must be made in several places;
- inconsistencies can appear;
- testing becomes harder;
- intent becomes less clear.

Instead:

```python
def standardize(values):
    mean = values.mean()
    sd = values.std()
    return (values - mean) / sd
```

A function gives a name to a computational idea.

---

# 2. Functions as abstractions

A useful mental model is:

```text
inputs
  ↓
function
  ↓
outputs
```

Example:

```python
def square(x):
    return x ** 2
```

A good function hides unnecessary implementation details while exposing a clear interface.

---

# 3. Function anatomy

```python
def mean_score(values):
    result = sum(values) / len(values)
    return result
```

Components:

```text
def             → function definition
mean_score      → function name
values          → parameter
result          → local variable
return          → output
```

---

# 4. Parameters vs. arguments

In:

```python
def standardize(values):
    ...
```

`values` is a **parameter**.

In:

```python
standardize(scores)
```

`scores` is an **argument**.

---

# 5. Return values

Prefer:

```python
def mean_score(values):
    return sum(values) / len(values)
```

rather than:

```python
def mean_score(values):
    print(sum(values) / len(values))
```

Returned values can be:

- assigned;
- tested;
- combined;
- reused downstream.

---

# 6. Multiple return values

```python
def summary(values):
    mean = sum(values) / len(values)
    minimum = min(values)
    maximum = max(values)

    return mean, minimum, maximum
```

Then:

```python
mean, minimum, maximum = summary(scores)
```

Python is effectively returning a tuple.

---

# 7. Default and keyword arguments

```python
def center(values, target=0):
    mean = sum(values) / len(values)
    return [
        x - mean + target
        for x in values
    ]
```

Call:

```python
center(scores)
```

or:

```python
center(scores, target=100)
```

Keyword arguments often make statistical code easier to read:

```python
simulate(
    n=1000,
    mean=0,
    sd=1,
    seed=556
)
```

---

# 8. Type hints

```python
def mean_score(
    values: list[float]
) -> float:
    return sum(values) / len(values)
```

Type hints improve:

- readability;
- documentation;
- IDE assistance;
- static analysis.

They make a function's interface more explicit.

---

# 9. Docstrings

```python
def mean_score(
    values: list[float]
) -> float:
    """Return the arithmetic mean of values."""
    return sum(values) / len(values)
```

Inspect:

```python
help(mean_score)
```

or:

```python
mean_score.__doc__
```

---

# 10. Pure functions

A **pure function** has two important properties:

1. The same inputs produce the same outputs.
2. It does not create observable side effects outside itself.

Example:

```python
def square(x):
    return x ** 2
```

Pure functions are often easier to reason about and test.

---

# 11. Side effects

A side effect changes something outside the function.

Examples:

- modifying a global variable;
- mutating an input;
- writing a file;
- printing;
- changing a database;
- making an API request.

Example:

```python
results = []

def add_result(x):
    results.append(x)
```

Calling `add_result()` changes external state.

---

# 12. Mutation as a side effect

From Week 2:

```python
def add_score(scores):
    scores.append(100)
```

A less surprising alternative:

```python
def add_score(scores):
    return scores + [100]
```

The second version leaves the input unchanged.

---

# 13. Pure does not mean universally better

Some tasks inherently involve side effects:

```text
reading data
writing a CSV
saving a plot
calling an API
updating a database
```

A useful design principle is:

> **Keep transformation logic as pure as practical, and isolate side effects near the boundaries of the workflow.**

```text
read data
   ↓
pure transformations
   ↓
summaries
   ↓
write output
```

---

# 14. Variable scope

**Scope** describes where a name can be accessed.

```python
def calculate():
    x = 10
    return x
```

Here, `x` is local to the function.

---

# 15. Local and global scope

Local:

```python
def double(x):
    result = x * 2
    return result
```

Global:

```python
RATE = 0.05

def apply_rate(x):
    return x * RATE
```

Global state can make dependencies less obvious.

More explicit:

```python
def apply_rate(x, rate):
    return x * rate
```

---

# 16. The LEGB rule

Python resolves names using:

```text
L → Local
E → Enclosing
G → Global
B → Built-in
```

For most statistical programming, the most important distinction is local vs. global scope.

---

# 17. Avoid unnecessary global state

Less explicit:

```python
threshold = 80

def classify(score):
    if score >= threshold:
        return "high"
    return "low"
```

Better:

```python
def classify(
    score,
    threshold=80
):
    if score >= threshold:
        return "high"

    return "low"
```

Now the dependency is part of the function interface.

---

# 18. Small functions and separation of concerns

Avoid one giant function:

```python
def do_everything(data):
    # clean
    # summarize
    # plot
    # write file
```

Prefer:

```python
clean_data(...)
summarize_data(...)
plot_results(...)
save_results(...)
```

A function should usually do one coherent thing.

---

# 19. Function composition

Small functions can be combined:

```python
clean = clean_data(raw)
filtered = filter_data(clean)
summary = summarize_data(filtered)
```

Conceptually:

```text
raw
 ↓
clean_data
 ↓
filter_data
 ↓
summarize_data
 ↓
summary
```

---

# 20. Functions are first-class objects

```python
def square(x):
    return x ** 2

f = square
f(5)
```

Functions can be:

- assigned to names;
- stored;
- passed to other functions;
- returned from functions.

This enables functional-programming patterns.

---

# 21. Higher-order functions

A higher-order function accepts another function or returns one.

```python
def apply_twice(
    function,
    value
):
    return function(
        function(value)
    )
```

Example:

```python
apply_twice(
    square,
    2
)
```

---

# 22. `map()`

`map()` applies a function to every item in an iterable.

```python
def square(x):
    return x ** 2

values = [1, 2, 3, 4]

result = map(
    square,
    values
)

list(result)
```

Result:

```text
[1, 4, 9, 16]
```

---

# 23. `map()` vs. comprehensions

Functional:

```python
list(
    map(
        square,
        values
    )
)
```

Comprehension:

```python
[
    square(x)
    for x in values
]
```

In Python, comprehensions are often clearer for simple transformations.

The conceptual value of `map()` is separating the transformation from iteration.

---

# 24. `filter()`

```python
def is_even(x):
    return x % 2 == 0
```

Then:

```python
result = filter(
    is_even,
    [1, 2, 3, 4, 5, 6]
)

list(result)
```

Result:

```text
[2, 4, 6]
```

A predicate decides which values survive.

---

# 25. `filter()` vs. comprehensions

Functional:

```python
list(
    filter(
        is_even,
        values
    )
)
```

Comprehension:

```python
[
    x
    for x in values
    if is_even(x)
]
```

Again, clarity matters more than using a particular construct.

---

# 26. `reduce()`

Import:

```python
from functools import reduce
```

Example:

```python
def add(x, y):
    return x + y

reduce(
    add,
    [1, 2, 3, 4]
)
```

Conceptually:

```text
((1 + 2) + 3) + 4
```

---

# 27. Reduction in statistics

Many summaries are reductions:

```text
sum
minimum
maximum
product
mean
```

A reduction transforms:

```text
many values
    ↓
one summary
```

Python's built-ins are usually preferable when available, but `reduce()` illustrates the underlying functional idea.

---

# 28. Lambda functions

A lambda is a small anonymous function:

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

Use lambdas when behavior is short and local.

Prefer a named function when the logic is complicated.

---

# 29. Functions with pandas

```python
def select_complete_cases(
    data,
    columns
):
    return data.dropna(
        subset=columns
    )
```

Then:

```python
complete = select_complete_cases(
    df,
    ["age", "score"]
)
```

This turns an analysis decision into reusable code.

---

# 30. Applying functions to Series

```python
def categorize_score(score):
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

---

# 31. Avoid hidden dependencies

Bad:

```python
df = ...

def summarize():
    return df["score"].mean()
```

Better:

```python
def summarize(data):
    return data["score"].mean()
```

The function's dependency is now explicit.

---

# 32. Functions as contracts

Think of a function as a contract:

```text
expects:
    DataFrame with score column

returns:
    float

guarantees:
    mean of nonmissing scores
```

Example:

```python
def mean_score(
    data: pd.DataFrame
) -> float:
    """Return mean of nonmissing scores."""
    return data["score"].mean()
```

---

# 33. Defensive checks

```python
def mean_score(data):
    if "score" not in data.columns:
        raise ValueError(
            "data must contain a score column"
        )

    return data["score"].mean()
```

This converts an assumption into an explicit check.

We will develop testing and exception handling more fully later.

---

# 34. Modules

As projects grow, reusable functions should move out of notebooks.

```text
project/
├── notebooks/
│   └── analysis.ipynb
├── src/
│   └── summaries.py
└── tests/
```

Inside `src/summaries.py`:

```python
def mean_score(...):
    ...

def summarize_group(...):
    ...
```

---

# 35. Importing your own functions

If:

```text
src/summaries.py
```

contains:

```python
def double(x):
    return x * 2
```

then:

```python
from src.summaries import double

double(10)
```

This separates reusable logic from exploratory analysis.

---

# 36. Why modularity matters

Modular code is easier to:

- reuse;
- test;
- debug;
- document;
- collaborate on;
- change safely.

Instead of:

```text
one 500-line notebook
```

prefer:

```text
small notebook
+
reusable functions
+
well-named modules
```

---

# 37. A statistical computing pipeline

```python
raw = load_data(...)
clean = clean_data(raw)
eligible = filter_population(clean)
summary = summarize(eligible)
figure = plot_results(eligible)
```

Each function has one responsibility.

The code reads like a description of the analysis.

---

# 38. Key ideas

By the end of Week 7, you should be able to explain:

1. Why functions improve statistical code.
2. Parameters vs. arguments.
3. Why return values are preferable to unnecessary printing.
4. Default and keyword arguments.
5. Type hints and docstrings.
6. What makes a function pure.
7. What a side effect is.
8. Why mutating inputs can be surprising.
9. Local vs. global scope.
10. The LEGB rule.
11. Why hidden global dependencies should be minimized.
12. Separation of concerns.
13. Function composition.
14. First-class and higher-order functions.
15. `map()`, `filter()`, and `reduce()`.
16. When comprehensions may be clearer.
17. Lambda functions.
18. How functions fit pandas workflows.
19. Defensive checks.
20. Why reusable functions belong in modules.

---

# 39. Recommended reading

## Python Tutorial — Defining Functions

https://docs.python.org/3/tutorial/controlflow.html#defining-functions

## Python Tutorial — More on Defining Functions

https://docs.python.org/3/tutorial/controlflow.html#more-on-defining-functions

## Python Documentation — Execution Model

https://docs.python.org/3/reference/executionmodel.html

## Python `functools`

https://docs.python.org/3/library/functools.html

---

# 40. YouTube recommendations

## 1. Engineering Digest — "Complete Python Functions Guide: From Basics to Advanced"

A broad functions tutorial covering function definitions, scope, lambda expressions, `map`, `filter`, `reduce`, and functions as first-class objects.

**Recommended use:** Use selected chapters rather than necessarily watching the entire video. The scope and functional-programming sections align particularly well with Week 7.

[Watch on YouTube](https://www.youtube.com/watch?v=vLEOBLDFq4g)

---

## 2. CampusX — "Lambda Functions in Python | Map, Filter and Reduce | Higher Order Functions in Python"

A focused treatment of lambda expressions, higher-order functions, and the `map` / `filter` / `reduce` family.

**Recommended use:** Watch alongside the functional-programming sections of these notes.

[Watch on YouTube](https://www.youtube.com/watch?v=ww2uPkwSjjY)

---

## 3. Python Scope / LEGB Rule

A focused scope tutorial is useful reinforcement for the distinction among Local, Enclosing, Global, and Built-in names.

**Recommended use:** Optional extension after the variable-scope exercises.

[Find LEGB tutorials on YouTube](https://www.youtube.com/results?search_query=Python+LEGB+scope+tutorial)

---

# 41. Week 7 takeaway

> **Functions turn a sequence of commands into a reusable computational vocabulary.**

```text
Repeated code
      ↓
Function
      ↓
Clear inputs + outputs
      ↓
Pure transformations
      ↓
Small composable components
      ↓
Modules
      ↓
Reusable statistical workflow
```

Next week we move into **Reproducibility & Reporting**, where modular code becomes part of literate and automated computational documents.
