# 1.2 Activity

## Python Objects, Data Structures, Mutability, Typing & Comprehensions

**Tools:** Python, VS Code and/or Jupyter

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

- inspect Python objects with `type()` and `id()`;
- distinguish names from objects;
- explain mutable vs. immutable objects;
- recognize shared references;
- use lists, tuples, dictionaries, and sets;
- explain dynamic typing;
- use basic type annotations;
- write list and dictionary comprehensions;
- connect data-structure choices to statistical computing.

## Part 0 — Set up

Use your Week 1 repository and create:

```text
notebooks/week02_data_structures.ipynb
```

or:

```text
src/week02_data_structures.py
```

## Part 1 — Everything is an object

Run:

```python
x = 42

type(x)
id(x)
repr(x)
```

Then:

```python
x = "hello"

type(x)
id(x)
repr(x)
```

### Questions

1. Did the type change?
2. Did the identity change?
3. What happened to the integer object?
4. Does the name `x` have a permanent type?

Write your answers in a Markdown cell.

## Part 2 — Names and objects

Run:

```python
x = [1, 2, 3]
y = x

x is y
```

Then:

```python
x.append(4)

print(x)
print(y)
```

### Prediction

Before running the code, predict the value of `y`.

Draw:

```text
x ─────┐
       ├────► ?
y ─────┘
```

Replace `?` with the object.

## Part 3 — Mutation vs. rebinding

Compare:

```python
x = [1, 2, 3]
y = x
x.append(4)
```

with:

```python
x = [1, 2, 3]
y = x
x = [1, 2, 3, 4]
```

For each, inspect:

```python
x
y
x is y
```

### Question

What is the difference between `x.append(4)` and `x = [1, 2, 3, 4]`?

## Part 4 — Mutable vs. immutable

Create:

```python
a = 10
b = 3.14
c = "hello"
d = (1, 2, 3)
e = [1, 2, 3]
f = {"a": 1}
g = {1, 2, 3}
```

Create a table:

| Object | Type | Mutable? |
|---|---|---|
| `a` | | |
| `b` | | |
| `c` | | |
| `d` | | |
| `e` | | |
| `f` | | |
| `g` | | |

Test your answers with operations such as:

```python
e.append(4)
f["b"] = 2
```

Try:

```python
d[0] = 99
```

Explain the result.

## Part 5 — A function that changes its input

Run:

```python
def add_value(values):
    values.append(100)

data = [1, 2, 3]
add_value(data)

print(data)
```

Then create a non-mutating version:

```python
def add_value(values):
    result = values.copy()
    result.append(100)
    return result

data = [1, 2, 3]
new_data = add_value(data)

print(data)
print(new_data)
```

### Reflection

Which style would you prefer for a statistical function, and why?

## Part 6 — Shallow copying

Run:

```python
x = [[1, 2], [3, 4]]
y = x.copy()

print(x is y)
print(x[0] is y[0])
```

Then:

```python
x[0].append(99)

print(x)
print(y)
```

Explain why the inner list changed in both objects.

## Part 7 — Lists

Create:

```python
scores = [72, 81, 91, 68, 88, 95, 77, 84]
```

Practice:

```python
scores[0]
scores[-1]
scores[2:5]
len(scores)
scores.append(90)
scores.sort()
scores.reverse()
```

### Challenge

Without NumPy, calculate:

- number of observations;
- minimum;
- maximum;
- mean.

## Part 8 — Tuples

Create:

```python
flagstaff = (35.1983, -111.6513)
```

Access:

```python
flagstaff[0]
flagstaff[1]
```

Try:

```python
flagstaff[0] = 40
```

Explain why a tuple could make conceptual sense for a coordinate pair.

## Part 9 — Dictionaries

Create:

```python
student = {
    "name": "Alex",
    "program": "Statistics and Data Science",
    "year": 1,
    "courses": ["STA 556", "STA 570"]
}
```

Practice:

```python
student["name"]
student["courses"]
student["graduation_year"] = 2028
"name" in student
student.get("email")
```

Why might `.get()` be preferable when a key may not exist?

## Part 10 — Sets

Create:

```python
states = [
    "Arizona", "Arizona", "Utah",
    "Nevada", "Arizona", "Utah"
]

unique_states = set(states)
```

Test:

```python
unique_states
"Arizona" in unique_states
"California" in unique_states
```

Explain why a set can be useful for unique categories and membership testing.

## Part 11 — Choosing data structures

Choose a data structure for each:

A. A sequence of ages where order matters.  
B. A fixed latitude/longitude coordinate.  
C. Subject ID → treatment group.  
D. Unique treatment groups.  
E. A collection that will be repeatedly modified.

Explain each answer.

## Part 12 — Dynamic typing

Run:

```python
x = 10
print(type(x))

x = "ten"
print(type(x))

x = [10]
print(type(x))
```

Then:

```python
x = "10"
x + 5
```

Explain why Python can be dynamically typed while still having types.

## Part 13 — Type annotations

Write:

```python
def mean(values: list[float]) -> float:
    return sum(values) / len(values)
```

Run:

```python
mean([1.0, 2.0, 3.0])
mean.__annotations__
```

Then:

```python
def add(a: int, b: int) -> int:
    return a + b

add(2, 3)
add("hello", "world")
```

Explain what the annotations communicate and why annotations do not make Python statically typed.

## Part 14 — List comprehensions

Traditional loop:

```python
squares = []

for x in range(10):
    squares.append(x ** 2)
```

Rewrite:

```python
squares = [x ** 2 for x in range(10)]
```

Then:

```python
even_squares = [
    x ** 2
    for x in range(21)
    if x % 2 == 0
]
```

Write the equivalent traditional loop and compare readability.

## Part 15 — Statistical example

Create:

```python
values = [12.1, 10.4, 15.2, 9.8, 13.7]
mean = sum(values) / len(values)

centered = [x - mean for x in values]
squared_deviations = [(x - mean) ** 2 for x in values]
```

Verify:

```python
sum(centered)
```

Then calculate the population variance from `squared_deviations`.

## Part 16 — Dictionary comprehensions

Run:

```python
squares = {x: x ** 2 for x in range(10)}
cubes = {x: x ** 3 for x in range(10)}
```

Create a dictionary mapping 1–10 to whether each number is even:

```python
even = {
    x: x % 2 == 0
    for x in range(1, 11)
}
```

## Part 17 — Structured observations

Create:

```python
observations = [
    {"id": 1, "score": 82},
    {"id": 2, "score": 91},
    {"id": 3, "score": 76},
    {"id": 4, "score": 88},
    {"id": 5, "score": 94},
]
```

Extract scores:

```python
scores = [obs["score"] for obs in observations]
```

Filter scores ≥ 85:

```python
high_scores = [
    obs["score"]
    for obs in observations
    if obs["score"] >= 85
]
```

Create ID → score:

```python
score_by_id = {
    obs["id"]: obs["score"]
    for obs in observations
}
```

### Challenge

Create a dictionary containing only observations with scores ≥ 85.

## Part 18 — When not to use a comprehension

Consider:

```python
results = []

for person in people:
    if person["age"] >= 18:
        if person["status"] == "active":
            transformed = complicated_function(person)
            results.append(transformed)
```

Ask whether compressing this into one comprehension would actually improve the code.

### Principle

> **Use comprehensions when they make simple transformations clearer.**

Do not use them merely to minimize lines.

## Part 19 — Mini data-science challenge

Create:

```python
participants = [
    {"id": 101, "age": 24, "group": "A", "score": 81},
    {"id": 102, "age": 31, "group": "B", "score": 94},
    {"id": 103, "age": 27, "group": "A", "score": 88},
    {"id": 104, "age": 19, "group": "B", "score": 72},
    {"id": 105, "age": 42, "group": "A", "score": 91},
    {"id": 106, "age": 35, "group": "B", "score": 85},
]
```

Using comprehensions, calculate:

```python
ids = ...
scores = ...
high_scores = ...
older_ids = ...
groups = ...
score_by_id = ...
```

Then create:

```python
summary = {
    "n": ...,
    "groups": ...,
    "mean_score": ...,
    "max_score": ...,
    "high_performers": ...
}
```

Add:

```python
summary: dict = ...
```

## Part 20 — Compare loops and comprehensions

Choose one previous problem and implement it both ways.

Answer:

1. Which is shorter?
2. Which is clearer?
3. Which would you prefer in production code?
4. Would your answer change if the transformation became more complicated?

## Part 21 — Git checkpoint

From your Week 1 repository:

```bash
git status
git diff
git add .
git commit -m "Complete Week 2 data structures exercises"
git push
```

Reinforce the Week 1 workflow:

```text
Create → Explore → Save → Inspect → Commit → Push
```

## Part 22 — Reflection

Answer these questions in a Markdown cell.

1. What is the relationship between a variable name and a Python object?
2. Why can mutability create unexpected behavior when passing objects to functions?
3. Why would you choose a dictionary instead of a list?
4. What does dynamically typed mean?
5. What problem can type annotations help solve?
6. When does a comprehension improve code?
7. Why should a statistician care about Python's object model and data structures?

## Completion checklist

- [ ] Created Week 2 notebook/script
- [ ] Inspected objects using `type()` and `id()`
- [ ] Demonstrated shared references
- [ ] Demonstrated mutation and rebinding
- [ ] Explored mutable and immutable objects
- [ ] Explored shallow copying
- [ ] Used lists, tuples, dictionaries, and sets
- [ ] Explained dynamic typing
- [ ] Added type annotations
- [ ] Created list comprehensions
- [ ] Created dictionary comprehensions
- [ ] Used comprehensions with conditions
- [ ] Completed the participant data challenge
- [ ] Compared loops with comprehensions
- [ ] Created a statistical summary object
- [ ] Committed Week 2 work to Git
- [ ] Pushed Week 2 work to GitHub

## What you should now understand

```text
Python names
      ↓
Objects
      ↓
Identity + type + value
      ↓
References
      ↓
Mutability
      ↓
Data structures
      ↓
Comprehensions
      ↓
Readable computational code
      ↓
Pandas DataFrames next
```
