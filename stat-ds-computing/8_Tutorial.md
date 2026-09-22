# 5.1 Activity

## Matrix Computation, Broadcasting, Vectorization, QR & SVD

**Tools:** Python, NumPy, Jupyter/VS Code

## Learning objectives

By the end of this tutorial, you should be able to:

- create and inspect NumPy arrays;
- reason about array shape;
- distinguish one-dimensional, row, and column vectors;
- replace simple Python loops with vectorized operations;
- apply broadcasting rules;
- use `axis` correctly;
- distinguish element-wise and matrix multiplication;
- solve linear systems numerically;
- compute and verify QR decompositions;
- compute and verify SVD decompositions;
- build low-rank approximations;
- connect SVD to dimensionality reduction;
- investigate conditioning and multicollinearity.

---

# Part 0 — Set up

Create:

```text
notebooks/week09_linear_algebra.ipynb
```

Import:

```python
import numpy as np
import pandas as pd
```

Create a reproducible random-number generator:

```python
rng = np.random.default_rng(556)
```

---

# Part 1 — Lists vs. arrays

```python
python_list = [1, 2, 3, 4]
python_list * 2
```

Now:

```python
x = np.array([1, 2, 3, 4])
x * 2
```

### Questions

1. What did multiplication mean for the list?
2. What did it mean for the array?
3. Which interpretation is more useful for numerical computing?

---

# Part 2 — Inspect array structure

```python
x = np.array([10, 20, 30])
```

Inspect:

```python
x.shape
x.ndim
x.dtype
x.size
```

Explain each output.

---

# Part 3 — Vector shapes

```python
x = np.array([1, 2, 3])
```

Create:

```python
x_col = x.reshape(-1, 1)
x_row = x.reshape(1, -1)
```

Inspect:

```python
x.shape
x_col.shape
x_row.shape
```

Explain why these shapes are different.

---

# Part 4 — Two-dimensional arrays

```python
X = np.array([
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9],
    [10, 11, 12]
])
```

Inspect:

```python
X.shape
X.ndim
X.size
```

Select:

```python
X[0, :]
X[:, 0]
X[1:3, :]
```

Connect this to row/column selection in pandas.

---

# Part 5 — Element-wise operations

Run:

```python
X + 10
X * 2
X ** 2
np.sqrt(X)
```

What does “element-wise” mean?

---

# Part 6 — Loop vs. vectorized computation

```python
values = list(range(1, 100001))
array = np.array(values)
```

Loop-style:

```python
squared_loop = [
    value ** 2
    for value in values
]
```

Vectorized:

```python
squared_vectorized = array ** 2
```

Verify:

```python
np.array_equal(
    np.array(squared_loop),
    squared_vectorized
)
```

---

# Part 7 — Benchmark

In Jupyter:

```python
%timeit [value ** 2 for value in values]
```

and:

```python
%timeit array ** 2
```

### Reflection

Which is faster, and why?

---

# Part 8 — Statistical vectorization

```python
scores = rng.normal(
    80,
    10,
    size=100
)
```

Calculate:

```python
mean = scores.mean()
sd = scores.std()

z = (
    scores - mean
) / sd
```

Verify:

```python
z.mean()
z.std()
```

Where is the iteration?

---

# Part 9 — Broadcasting a scalar

```python
X = np.array([
    [1, 2, 3],
    [4, 5, 6]
])

X + 10
```

Write the conceptual matrix that would produce the same result.

---

# Part 10 — Broadcasting across rows

```python
v = np.array([10, 20, 30])

X + v
```

Inspect:

```python
X.shape
v.shape
```

Why are the shapes compatible?

---

# Part 11 — Broadcasting failure

```python
bad = np.array([10, 20])
```

Try:

```python
X + bad
```

Read the error and explain why the shapes are incompatible.

---

# Part 12 — Fix the broadcasting problem

```python
bad_col = bad.reshape(-1, 1)

X + bad_col
```

Inspect:

```python
bad_col.shape
```

How did reshaping change the meaning of the operation?

---

# Part 13 — Understanding `axis`

```python
X = np.array([
    [10, 20],
    [12, 25],
    [14, 30]
])
```

Predict the shape, then calculate:

```python
X.mean(axis=0)
X.mean(axis=1)
```

---

# Part 14 — Column standardization

```python
means = X.mean(axis=0)
sds = X.std(axis=0)

Z = (
    X - means
) / sds
```

Verify:

```python
Z.mean(axis=0)
Z.std(axis=0)
```

Which part depends on broadcasting?

---

# Part 15 — `keepdims`

Compare:

```python
means_a = X.mean(axis=0)
```

with:

```python
means_b = X.mean(
    axis=0,
    keepdims=True
)
```

Inspect shapes.

---

# Part 16 — Element-wise vs. matrix multiplication

```python
A = np.array([
    [1, 2],
    [3, 4]
])

B = np.array([
    [5, 6],
    [7, 8]
])
```

Compare:

```python
A * B
A @ B
```

Write out the first element of each result manually.

---

# Part 17 — Shape rules for matrix multiplication

```python
A = rng.normal(size=(5, 3))
B = rng.normal(size=(3, 2))
```

Predict:

```python
(A @ B).shape
```

Then create an incompatible matrix and explain the resulting error.

---

# Part 18 — Dot products

```python
x = np.array([1, 2, 3])
y = np.array([4, 5, 6])

x @ y
```

Verify manually.

---

# Part 19 — Model predictions

```python
X = np.array([
    [1.0, 2.0],
    [2.0, 1.0],
    [3.0, 4.0],
    [4.0, 3.0]
])

beta = np.array([2.0, -1.0])
beta_0 = 5.0

y_hat = beta_0 + X @ beta
```

If the leading space causes an indentation error, fix it.

Identify where broadcasting occurs.

---

# Part 20 — Solve a linear system

```python
A = np.array([
    [3.0, 1.0],
    [1.0, 2.0]
])

b = np.array([9.0, 8.0])

x = np.linalg.solve(A, b)
```

Verify:

```python
A @ x
```

---

# Part 21 — Avoid explicit inversion

```python
x_inverse = np.linalg.inv(A) @ b
```

Compare with `x`.

Why might `solve()` still be preferable?

---

# Part 22 — Build a regression design matrix

```python
n = 100

x1 = rng.normal(size=n)
x2 = rng.normal(size=n)

X = np.column_stack([
    np.ones(n),
    x1,
    x2
])
```

Inspect:

```python
X.shape
X[:5]
```

Explain the first column.

---

# Part 23 — Simulate a response

```python
beta_true = np.array([
    5.0,
    2.0,
    -1.5
])

y = (
    X @ beta_true
    + rng.normal(0, 1, size=n)
)
```

Fix any accidental indentation if necessary.

Which operation creates the systematic component of the model?

---

# Part 24 — Least squares

```python
beta_hat, residuals, rank, singular_values = (
    np.linalg.lstsq(
        X,
        y,
        rcond=None
    )
)
```

Compare:

```python
beta_hat
beta_true
```

Why are they close but not identical?

---

# Part 25 — QR decomposition

```python
Q, R = np.linalg.qr(
    X,
    mode="reduced"
)
```

Inspect shapes and verify:

```python
np.allclose(
    X,
    Q @ R
)
```

---

# Part 26 — Verify orthogonality

```python
Q.T @ Q
```

Then:

```python
np.allclose(
    Q.T @ Q,
    np.eye(Q.shape[1])
)
```

What mathematical property are you checking?

---

# Part 27 — Least squares using QR

```python
beta_qr = np.linalg.solve(
    R,
    Q.T @ y
)
```

Compare:

```python
beta_qr
beta_hat
```

Check:

```python
np.allclose(
    beta_qr,
    beta_hat
)
```

---

# Part 28 — SVD

```python
M = rng.normal(size=(6, 4))

U, s, Vt = np.linalg.svd(
    M,
    full_matrices=False
)
```

Inspect:

```python
U.shape
s.shape
Vt.shape
s
```

---

# Part 29 — Reconstruct from SVD

```python
S = np.diag(s)

M_reconstructed = (
    U
    @ S
    @ Vt
)
```

Check:

```python
np.allclose(
    M,
    M_reconstructed
)
```

Why use `allclose()` instead of `==`?

---

# Part 30 — Low-rank approximation

```python
k = 2

M_k = (
    U[:, :k]
    @ np.diag(s[:k])
    @ Vt[:k, :]
)
```

Calculate error:

```python
error = np.linalg.norm(
    M - M_k
)

error
```

---

# Part 31 — Approximation error vs. rank

Repeat for:

```text
k = 1, 2, 3, 4
```

Store the errors.

What happens as more singular components are retained?

---

# Part 32 — SVD on correlated data

```python
x1 = rng.normal(size=200)

x2 = (
    2 * x1
    + rng.normal(
        0,
        0.1,
        size=200
    )
)

x3 = rng.normal(size=200)

X_corr = np.column_stack([
    x1,
    x2,
    x3
])

X_centered = (
    X_corr
    - X_corr.mean(axis=0)
)
```

Compute:

```python
U, s, Vt = np.linalg.svd(
    X_centered,
    full_matrices=False
)

s
```

Does one direction appear weaker than the others?

---

# Part 33 — Condition number

```python
np.linalg.cond(X_centered)
```

Compare with:

```python
X_independent = rng.normal(
    size=(200, 3)
)

np.linalg.cond(X_independent)
```

How does this connect to multicollinearity?

---

# Part 34 — Matrix rank

```python
np.linalg.matrix_rank(X_centered)
```

Now:

```python
X_exact = np.column_stack([
    x1,
    2 * x1,
    x3
])

np.linalg.matrix_rank(X_exact)
```

Explain the result.

---

# Part 35 — Vectorized distance challenge

```python
points = rng.normal(
    size=(1000, 2)
)

target = np.array([
    0.5,
    -0.5
])
```

Compute all Euclidean distances without a Python loop:

```python
distances = np.sqrt(
    (
        (points - target) ** 2
    ).sum(axis=1)
)
```

Identify where broadcasting, element-wise computation, and reduction occur.

---

# Part 36 — Loop version

Implement the same distance calculation with a Python loop.

Verify:

```python
np.allclose(
    distances_loop,
    distances
)
```

Benchmark both versions.

---

# Part 37 — Mini statistical-computing challenge

Simulate:

```python
n = 500

X = rng.normal(
    size=(n, 5)
)

beta = np.array([
    2.0,
    -1.0,
    0.5,
    3.0,
    -2.0
])

y = (
    4
    + X @ beta
    + rng.normal(
        0,
        2,
        size=n
    )
)
```

Fix any accidental indentation before execution.

Complete:

1. Standardize all predictor columns using broadcasting.
2. Create a design matrix with an intercept.
3. Estimate coefficients using `np.linalg.lstsq()`.
4. Compute fitted values.
5. Compute residuals.
6. Compute residual sum of squares.
7. Compute a QR decomposition.
8. Verify QR reconstruction.
9. Compute singular values.
10. Compute the condition number.

---

# Part 38 — Compare implementations

For one computation, implement:

### Version A

Explicit Python loop.

### Version B

Vectorized NumPy expression.

Compare:

- clarity;
- number of lines;
- speed;
- correspondence with mathematical notation.

---

# Part 39 — Numerical validation

For every decomposition, ask:

### Shape

Do the matrix dimensions make sense?

### Reconstruction

Does `Q @ R` recover `X`?

Does `U @ S @ Vt` recover the matrix?

### Orthogonality

Does `Q.T @ Q` approximate the identity matrix?

### Tolerance

Are numerical comparisons performed with:

```python
np.allclose()
```

rather than exact equality?

---

# Part 40 — Git checkpoint

```bash
git status
git add .
git commit -m "Complete Week 9 matrix computation exercises"
git push
```

---

# Part 41 — Final reflection

### 1. Arrays

Why are NumPy arrays better suited than Python lists for many numerical computations?

### 2. Shape

Why is `(n,)` different from `(n, 1)`?

### 3. Vectorization

What does vectorization mean?

### 4. Broadcasting

Explain broadcasting in your own words.

### 5. Axis

What does `axis=0` mean when calculating a matrix mean?

### 6. Multiplication

What is the difference between `A * B` and `A @ B`?

### 7. Linear systems

Why prefer `solve()` to explicitly calculating a matrix inverse?

### 8. QR

What are the main properties of `Q` and `R`?

### 9. SVD

What do singular values tell us?

### 10. Statistics

How can SVD or conditioning reveal problems such as multicollinearity?

---

# Completion checklist

- [ ] Created Week 9 notebook
- [ ] Created NumPy arrays
- [ ] Inspected shape, dimension, size, and dtype
- [ ] Compared `(n,)`, `(n,1)`, and `(1,n)`
- [ ] Used element-wise operations
- [ ] Compared loops and vectorization
- [ ] Benchmarked vectorized operations
- [ ] Standardized data using vectorization
- [ ] Applied scalar broadcasting
- [ ] Applied row/column broadcasting
- [ ] Diagnosed a broadcasting error
- [ ] Used `axis=0` and `axis=1`
- [ ] Used `keepdims=True`
- [ ] Distinguished element-wise from matrix multiplication
- [ ] Used matrix multiplication shape rules
- [ ] Computed dot products
- [ ] Generated model predictions with matrix algebra
- [ ] Solved a linear system
- [ ] Compared solving with explicit inversion
- [ ] Built a regression design matrix
- [ ] Estimated least squares with `lstsq`
- [ ] Computed QR decomposition
- [ ] Verified QR reconstruction
- [ ] Verified orthogonality
- [ ] Solved least squares using QR
- [ ] Computed SVD
- [ ] Reconstructed a matrix from SVD
- [ ] Built a low-rank approximation
- [ ] Measured approximation error
- [ ] Explored correlated predictors
- [ ] Calculated a condition number
- [ ] Calculated matrix rank
- [ ] Implemented vectorized distances
- [ ] Compared loop and vectorized implementations
- [ ] Completed the statistical-computing challenge
- [ ] Used `np.allclose()` for numerical checks
- [ ] Committed Week 9 work to Git
- [ ] Pushed work to GitHub

---

# What you should now understand

```text
Statistical problem
       ↓
Array representation
       ↓
Inspect shapes
       ↓
Vectorize
       ↓
Broadcast
       ↓
Matrix multiplication
       ↓
Numerical linear algebra
       ↓
QR / SVD
       ↓
Stable and efficient computation
```

Next week we will use this numerical foundation for **random-number generation, inverse-transform sampling, and Monte Carlo simulation**.
