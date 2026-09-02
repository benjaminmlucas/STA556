# 1.2 Advanced Data Types & Structures

## Why this week matters

Week 1 established the professional computational workflow. Week 2 moves inside Python and asks:

> **What actually happens when Python stores and manipulates data?**

The syllabus identifies Week 2 as **Advanced Data Types & Structures**, covering mutable vs. immutable objects, memory models, static vs. dynamic typing, and list/dictionary comprehensions.

For statistical computing, these ideas matter because data representation affects how code behaves, how functions interact with their inputs, readability, and eventually computational efficiency.

## 1. Everything in Python is an object

Python's data model describes objects as the fundamental abstraction for data. Every object has an **identity**, **type**, and **value**.

```python
x = 42

type(x)
id(x)
repr(x)
```

A useful mental model is:

```text
x ─────► Python object
          type: int
          value: 42
          identity: ...
```

A variable is best thought of as a **name bound to an object**, rather than a box permanently containing a value.

## 2. Identity, type, and value

### Identity

```python
id(x)
x is y
```

`is` compares object identity.

### Type

```python
type(x)
```

The type determines what operations an object supports.

### Value

The value is the information represented by the object.

These three concepts become especially important when multiple names refer to the same object.

## 3. Names and references

Consider:

```python
x = [1, 2, 3]
y = x
```

Now:

```python
x is y
```

returns `True`.

Conceptually:

```text
x ─────┐
       ├────► [1, 2, 3]
y ─────┘
```

There is one list object and two names referring to it.

## 4. Mutable vs. immutable objects

The Python data model distinguishes objects whose values can change from objects whose values cannot change after creation.

Common immutable types:

- `int`
- `float`
- `bool`
- `str`
- `tuple`

Common mutable types:

- `list`
- `dict`
- `set`

The key question is:

> **Can this object be changed in place?**

## 5. Mutation

```python
x = [1, 2, 3]
y = x

x.append(4)

print(x)
print(y)
```

Both names now refer to:

```text
[1, 2, 3, 4]
```

because the list itself was mutated.

This matters in statistical functions. If a function receives a mutable object and changes it, the caller may see that change.

## 6. Rebinding is different from mutation

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

The first changes the shared list. The second changes what `x` refers to.

This distinction—**mutating an object vs. rebinding a name**—is fundamental to understanding Python.

## 7. Copies

```python
x = [1, 2, 3]
y = x.copy()

x is y
```

returns `False`.

But nested structures introduce shallow vs. deep copying:

```python
x = [[1, 2], [3, 4]]
y = x.copy()

x[0].append(99)

print(x)
print(y)
```

The outer lists differ, but the inner lists are shared.

## 8. Python's major collection types

| Type | Ordered? | Mutable? | Typical use |
|---|---|---|---|
| `list` | Yes | Yes | General sequence |
| `tuple` | Yes | No | Fixed collection |
| `dict` | Yes* | Yes | Key-value mapping |
| `set` | No | Yes | Unique values/membership |

Modern Python dictionaries preserve insertion order. Dictionaries support key lookup, membership, deletion, and comprehensions.

### List

```python
scores = [72, 81, 91, 68]
scores.append(95)
scores[1:3]
```

### Tuple

```python
coordinate = (35.1983, -111.6513)
```

A tuple is immutable.

### Dictionary

```python
student = {
    "name": "Alex",
    "program": "Statistics and Data Science",
    "year": 1
}

student["name"]
student.get("email")
```

### Set

```python
states = {"Arizona", "Utah", "Nevada"}
```

Sets are useful for uniqueness and membership testing.

## 9. Choosing a data structure

Ask:

> **What relationship am I trying to represent?**

Sequence → `list`

Fixed collection → `tuple`

Key-value relationship → `dict`

Unique membership → `set`

Data structures are not merely containers. They influence which operations are natural and efficient.

## 10. Python is dynamically typed

Python is a **dynamically typed** language: types are checked during execution rather than requiring the whole program to pass a static type check before running.

For example:

```python
x = 10
x = "hello"
```

is valid.

But Python is not "untyped":

```python
x = "10"
x + 5
```

produces a runtime type error.

The values still have types; the type relationship is handled dynamically.

## 11. Type annotations

Python remains dynamically typed, but it supports annotations that can be used by static analysis tools.

```python
def mean(values: list[float]) -> float:
    return sum(values) / len(values)
```

Annotations improve:

- readability;
- documentation;
- IDE assistance;
- static analysis;
- collaboration.

Important:

> **Type hints do not turn Python into a statically typed language.**

## 12. List comprehensions

Traditional loop:

```python
squares = []

for x in range(10):
    squares.append(x ** 2)
```

List comprehension:

```python
squares = [x ** 2 for x in range(10)]
```

General form:

```python
[expression for item in iterable]
```

With a condition:

```python
[x ** 2 for x in range(10) if x % 2 == 0]
```

The goal is not simply fewer lines. Use comprehensions when they make a simple transformation clearer.

## 13. Dictionary comprehensions

```python
squares = {x: x ** 2 for x in range(5)}
```

General form:

```python
{key_expression: value_expression for item in iterable}
```

Dictionary comprehensions are standard Python syntax.

## 14. Statistical computing example

```python
values = [12.1, 10.4, 15.2, 9.8, 13.7]

mean = sum(values) / len(values)

centered = [x - mean for x in values]
squared_deviations = [(x - mean) ** 2 for x in values]
```

This illustrates how Python's built-in structures can represent simple statistical transformations. Later we will use NumPy for more efficient vectorized computation.

## 15. Structured observations

```python
observations = [
    {"id": 1, "score": 82},
    {"id": 2, "score": 91},
    {"id": 3, "score": 76},
    {"id": 4, "score": 88},
]
```

Extract scores:

```python
scores = [obs["score"] for obs in observations]
```

Filter:

```python
high_scores = [
    obs["score"]
    for obs in observations
    if obs["score"] >= 85
]
```

Create a mapping:

```python
score_by_id = {
    obs["id"]: obs["score"]
    for obs in observations
}
```

This pattern provides a useful bridge from Python's built-in structures to real-world tabular data.

## 16. When not to use comprehensions

Do not optimize for the fewest lines.

A complicated comprehension such as:

```python
result = [f(g(h(x))) for x in data if complicated_condition(x)]
```

may be less readable than a normal loop.

Professional code prioritizes **clarity, correctness, and maintainability**.

## 17. Connection to later course material

The progression is:

```text
Objects
  ↓
Types
  ↓
Identity + references
  ↓
Mutability
  ↓
Data structures
  ↓
Comprehensions
  ↓
Algorithms
  ↓
Efficient statistical computing
```

These concepts prepare us for DataFrames, vectorization, matrix computation, simulation, optimization, testing, and profiling.

## 18. Key ideas

By the end of Week 2, students should be able to explain:

1. A Python variable is a name bound to an object.
2. Objects have identity, type, and value.
3. Mutable objects can change in place.
4. Immutable objects cannot be changed in place.
5. Multiple names can refer to the same object.
6. Python is dynamically typed.
7. Python still has a formal type system.
8. Type annotations can support static analysis.
9. Lists, tuples, dictionaries, and sets represent different relationships.
10. Comprehensions provide concise collection-building syntax.
11. Data-structure choices affect readable and efficient computational code.

## 19. Recommended reading

### Python Data Model

Official reference for objects, identity, types, values, mutability, and references.

https://docs.python.org/3/reference/datamodel.html

### Python Tutorial — Data Structures

Practical reference for lists, tuples, sets, dictionaries, and comprehensions.

https://docs.python.org/3/tutorial/datastructures.html

### Python Typing Documentation

Official documentation for static, dynamic, and gradual typing and type annotations.

https://typing.python.org/en/latest/

### Python Data Science Handbook

Jake VanderPlas provides context for how Python's data structures connect to NumPy and the scientific Python ecosystem.

https://jakevdp.github.io/PythonDataScienceHandbook/


## 21. YouTube recommendations

### 1. Corey Schafer — Mutable vs Immutable

A short, focused explanation of the difference between mutable and immutable Python objects. This is particularly useful alongside the Week 2 exercises on mutation, references, and copying.

**Recommended use:** Watch before or after **Part 5 — A function that changes its input** in the tutorial.

[Watch on YouTube](https://www.youtube.com/watch?v=5qQQ3yzbKp8)

### 2. Corey Schafer — Python Comprehensions

A practical explanation of list, dictionary, and set comprehensions, including why comprehensions can be preferable to traditional loops for simple transformations.

**Recommended use:** Watch before **Parts 14–17** of the tutorial.

[Watch on YouTube](https://www.youtube.com/watch?v=3dt4OGnU5sM)

### 3. Real Python — Dynamic vs Static Typing

A concise explanation of dynamic versus static typing, including examples of when Python performs type checking and how type hints can add static-analysis capabilities.

**Recommended use:** Watch alongside **Section 10 — Python is dynamically typed**.

[Watch the video](https://realpython.com/videos/dynamic-vs-static/)

### 4. Real Python — Type Hinting

An introduction to Python type annotations, including function arguments and return values. It provides a useful extension of the basic type-hinting material in this week's tutorial.

**Recommended use:** Optional extension after **Part 13 — Type annotations**.

[Watch the video](https://realpython.com/lessons/type-hinting/)


## 20. Week 2 takeaway

> **Good statistical computing begins with understanding how data are represented and manipulated.**

The next step is:

```text
Python structures
      ↓
Pandas DataFrames
      ↓
Indexing
      ↓
Slicing
      ↓
Filtering
      ↓
Real-world data
```
