# 5.1 Matrix Computation & Linear Algebra

## Why this week matters

Up to this point, STA 556 has focused on data engineering, visualization, reproducibility, and software design. Week 9 shifts toward the numerical core of statistical computing.

The central question is:

> **How can we represent statistical computations as efficient operations on vectors and matrices?**

This week focuses on:

- NumPy arrays;
- vectorized computation;
- broadcasting;
- matrix multiplication;
- linear systems;
- QR decomposition;
- Singular Value Decomposition (SVD);
- statistical applications of matrix decompositions.

These ideas underlie regression, optimization, simulation, machine learning, and numerical statistics.

---

# 1. Why arrays matter

Python lists are flexible:

```python
x = [1, 2, 3, 4]
```

but numerical computing usually benefits from NumPy arrays:

```python
import numpy as np

x = np.array([1, 2, 3, 4])
```

Arrays provide:

- homogeneous numerical storage;
- multidimensional structure;
- vectorized operations;
- broadcasting;
- optimized numerical routines;
- linear algebra functions.

---

# 2. Shape is part of the data

Create:

```python
x = np.array([1, 2, 3])
```

Inspect:

```python
x.shape
x.ndim
x.dtype
```

A matrix:

```python
X = np.array([
    [1, 2],
    [3, 4],
    [5, 6]
])
```

has shape:

```text
(3, 2)
```

meaning three rows and two columns.

> **In numerical computing, shape is part of the meaning of an object.**

---

# 3. One-dimensional arrays can be subtle

```python
x = np.array([1, 2, 3])
```

has shape:

```text
(3,)
```

not:

```text
(3, 1)
```

Create a column vector:

```python
x_col = x.reshape(-1, 1)
```

and a row vector:

```python
x_row = x.reshape(1, -1)
```

These distinctions matter for broadcasting and matrix multiplication.

---

# 4. Element-wise operations

NumPy arithmetic is usually element-wise.

```python
x = np.array([1, 2, 3])

x + 10
x * 2
x ** 2
```

This differs from ordinary Python-list behavior.

---

# 5. Vectorization

Loop-based code:

```python
result = []

for value in x:
    result.append(value ** 2)
```

Vectorized code:

```python
result = x ** 2
```

Vectorization means expressing the operation in array form rather than manually controlling iteration in Python.

---

# 6. Why vectorization matters

Vectorization can improve:

- readability;
- concision;
- speed;
- correspondence with mathematical notation.

For example:

```python
z = (x - x.mean()) / x.std()
```

closely resembles:

```text
z_i = (x_i - x̄) / s
```

Vectorized code often allows the implementation to resemble the mathematics.

---

# 7. Vectorization does not mean “no loops”

Array operations still involve iteration internally.

The difference is where the loop runs.

A Python loop executes at the Python level. NumPy operations often delegate work to optimized compiled numerical code.

> **Vectorization means expressing operations in array form so optimized numerical routines can perform the iteration.**

---

# 8. Broadcasting

Broadcasting allows NumPy to perform operations between arrays with compatible but different shapes.

```python
X = np.array([
    [1, 2, 3],
    [4, 5, 6]
])

X + 10
```

The scalar behaves as though it were applied to every element.

---

# 9. Broadcasting a vector across rows

```python
X = np.array([
    [1, 2, 3],
    [4, 5, 6]
])

v = np.array([10, 20, 30])

X + v
```

returns:

```text
[[11, 22, 33],
 [14, 25, 36]]
```

Shapes:

```text
X: (2, 3)
v:    (3,)
```

The vector is broadcast across rows.

---

# 10. Broadcasting rules

NumPy compares dimensions from the **rightmost dimension moving left**.

Two dimensions are compatible when:

1. they are equal; or
2. one of them is 1.

Compatible:

```text
(4, 3)
   (3,)
```

and:

```text
(4, 3)
(4, 1)
```

Understanding shapes is often the key to debugging broadcasting errors.

---

# 11. Column-wise centering

```python
X = np.array([
    [10, 20],
    [12, 25],
    [14, 30]
])

means = X.mean(axis=0)
X_centered = X - means
```

Broadcasting subtracts each column mean from the appropriate column.

This is a common statistical preprocessing operation expressed in one line.

---

# 12. Understanding `axis`

For a matrix:

```python
X.mean(axis=0)
```

collapses rows and returns one mean per **column**.

```python
X.mean(axis=1)
```

collapses columns and returns one mean per **row**.

> `axis` tells NumPy which dimension to collapse.

---

# 13. Keep dimensions when useful

```python
X.mean(
    axis=0,
    keepdims=True
)
```

Without `keepdims=True`, the shape might be:

```text
(p,)
```

With it:

```text
(1, p)
```

Preserving dimensions can make broadcasting behavior more explicit.

---

# 14. Element-wise multiplication vs. matrix multiplication

Element-wise:

```python
A * B
```

Matrix multiplication:

```python
A @ B
```

These are fundamentally different operations.

---

# 15. Matrix multiplication

If:

```text
A is m × n
B is n × p
```

then:

```text
A @ B
```

has shape:

```text
m × p
```

The inner dimensions must match.

---

# 16. Dot products

```python
x = np.array([1, 2, 3])
y = np.array([4, 5, 6])

x @ y
```

returns:

```text
32
```

because:

```text
1×4 + 2×5 + 3×6 = 32
```

The dot product is central to projections, regression, and machine learning.

---

# 17. Statistical design matrices

Linear regression can be written:

```text
y = Xβ + ε
```

where:

```text
y → n × 1 response vector
X → n × p design matrix
β → p × 1 coefficient vector
```

In NumPy:

```python
predictions = X @ beta
```

This directly translates the statistical model into code.

---

# 18. Avoid explicit matrix inversion

OLS is often written:

```text
β̂ = (XᵀX)⁻¹Xᵀy
```

It is tempting to code:

```python
beta = (
    np.linalg.inv(X.T @ X)
    @ X.T
    @ y
)
```

But explicit inversion is usually not the preferred numerical approach.

Instead:

```python
beta, *_ = np.linalg.lstsq(
    X,
    y,
    rcond=None
)
```

Numerical algorithms matter.

---

# 19. Linear systems

For:

```text
Ax = b
```

use:

```python
x = np.linalg.solve(A, b)
```

rather than:

```python
np.linalg.inv(A) @ b
```

when solving a linear system.

---

# 20. Orthogonality

Vectors `u` and `v` are orthogonal when:

```text
uᵀv = 0
```

In NumPy:

```python
u @ v
```

Orthogonal directions do not overlap in their linear information, which makes orthogonal decompositions especially useful numerically.

---

# 21. QR decomposition

The QR decomposition writes:

```text
X = QR
```

where:

```text
Q → orthonormal columns
R → upper triangular matrix
```

In NumPy:

```python
Q, R = np.linalg.qr(X)
```

---

# 22. Check a QR decomposition

Verify reconstruction:

```python
np.allclose(
    X,
    Q @ R
)
```

Check orthogonality:

```python
np.allclose(
    Q.T @ Q,
    np.eye(Q.shape[1])
)
```

This turns mathematical properties into computational checks.

---

# 23. QR and least squares

QR provides a stable route to least squares without explicitly forming a matrix inverse.

If:

```text
X = QR
```

then the problem:

```text
min ||y - Xβ||²
```

can be transformed using the orthogonality of `Q`.

The broader lesson is:

> **Matrix decompositions turn numerical problems into structured subproblems.**

---

# 24. Singular Value Decomposition

For an `m × n` matrix:

```text
X = U Σ Vᵀ
```

where:

```text
U → left singular vectors
Σ → singular values
V → right singular vectors
```

In NumPy:

```python
U, s, Vt = np.linalg.svd(
    X,
    full_matrices=False
)
```

NumPy returns singular values in a one-dimensional array `s`.

---

# 25. Interpreting the SVD

A useful geometric interpretation is:

```text
Vᵀ
 ↓
re-express input directions

Σ
 ↓
stretch or shrink

U
 ↓
re-express output directions
```

Singular values quantify the strength of the corresponding directions.

---

# 26. Singular values

```python
U, s, Vt = np.linalg.svd(
    X,
    full_matrices=False
)

s
```

Singular values are returned from largest to smallest.

Large values indicate strong directions in the matrix; very small values indicate weak or nearly redundant directions.

---

# 27. Reconstruct the matrix

```python
S = np.diag(s)

X_reconstructed = (
    U
    @ S
    @ Vt
)
```

Check:

```python
np.allclose(
    X,
    X_reconstructed
)
```

---

# 28. Low-rank approximation

Retain only the first `k` singular components:

```python
k = 2

X_k = (
    U[:, :k]
    @ np.diag(s[:k])
    @ Vt[:k, :]
)
```

Conceptually:

```text
original matrix
      ↓
retain strongest directions
      ↓
compressed approximation
```

---

# 29. SVD and dimensionality reduction

For centered data, dominant singular directions identify important patterns in the observation-by-variable matrix.

This is closely related to Principal Component Analysis (PCA).

We will not develop PCA fully this week, but SVD provides much of its computational foundation.

---

# 30. Rank

The rank of a matrix is the number of linearly independent directions it contains.

```python
np.linalg.matrix_rank(X)
```

From an SVD perspective, rank relates to the number of non-negligible singular values.

---

# 31. Conditioning

A matrix may be mathematically invertible yet numerically difficult to work with.

```python
np.linalg.cond(X)
```

A large condition number indicates sensitivity to small perturbations.

In statistical models, this can occur when predictors are nearly linearly dependent.

---

# 32. Multicollinearity

If:

```text
x₂ ≈ 2x₁
```

then the design matrix contains nearly redundant information.

Possible consequences:

- unstable coefficient estimates;
- large condition numbers;
- small singular values;
- sensitivity to numerical error.

Linear algebra reveals the structure behind multicollinearity.

---

# 33. Broadcasting and standardization

```python
means = X.mean(axis=0)
sds = X.std(axis=0)

Z = (
    X - means
) / sds
```

No explicit Python loop is required.

---

# 34. Broadcasting and model predictions

If:

```text
X shape = (n, p)
beta shape = (p,)
```

then:

```python
X @ beta
```

has shape:

```text
(n,)
```

Adding an intercept:

```python
predictions = beta_0 + X @ beta
```

uses scalar broadcasting.

---

# 35. Vectorized distances

Suppose rows of `X` are observations and:

```python
target = np.array([2, 3])
```

Then:

```python
distances = np.sqrt(
    ((X - target) ** 2)
    .sum(axis=1)
)
```

combines:

- broadcasting;
- element-wise arithmetic;
- reduction across an axis.

These patterns occur frequently in statistical algorithms.

---

# 36. Loops are not forbidden

A Python loop is appropriate when:

- operations are inherently sequential;
- array formulation is unnatural;
- vectorization would harm clarity.

The goal is not:

> “Never use loops.”

The goal is:

> **Recognize when numerical structure allows a clearer and faster array formulation.**

---

# 37. Benchmark thoughtfully

In Jupyter:

```python
%timeit [x ** 2 for x in values]
```

versus:

```python
%timeit array ** 2
```

Performance depends on problem size and implementation.

Do not optimize without measuring.

---

# 38. Numerical correctness

Floating-point arithmetic is approximate.

Avoid exact equality checks for many numerical matrix computations:

```python
A == B
```

Prefer:

```python
np.allclose(A, B)
```

This compares values within numerical tolerances.

---

# 39. Numerical-computing workflow

```text
Identify mathematical structure
        ↓
Represent data as arrays
        ↓
Inspect shapes
        ↓
Vectorize operations
        ↓
Use broadcasting deliberately
        ↓
Use established linear algebra routines
        ↓
Check decompositions
        ↓
Validate numerical results
```

---

# 40. Key ideas

By the end of Week 9, you should be able to explain:

1. Why NumPy arrays are central to statistical computing.
2. Why array shape matters.
3. `(n,)` vs. `(n, 1)` vs. `(1, n)`.
4. Element-wise operations.
5. Vectorization.
6. Why vectorized code can outperform Python loops.
7. Broadcasting rules.
8. How `axis` works.
9. Element-wise vs. matrix multiplication.
10. Matrix-multiplication dimension rules.
11. Why solvers are preferable to explicit inversion.
12. Orthogonality.
13. QR decomposition.
14. QR and least squares.
15. SVD.
16. Singular values.
17. Low-rank approximation.
18. SVD and dimensionality reduction.
19. Matrix rank and conditioning.
20. Multicollinearity from a linear-algebra perspective.
21. Why numerical comparisons use tolerances.

---

# 41. Recommended reading

## NumPy — Broadcasting

https://numpy.org/doc/stable/user/basics.broadcasting.html

## NumPy — Linear Algebra

https://numpy.org/doc/stable/reference/routines.linalg.html

## Python Data Science Handbook — NumPy

https://jakevdp.github.io/PythonDataScienceHandbook/

## NumPy — `linalg.svd`

https://numpy.org/doc/stable/reference/generated/numpy.linalg.svd.html

## NumPy — `linalg.qr`

https://numpy.org/doc/stable/reference/generated/numpy.linalg.qr.html

---

# 42. YouTube recommendations

## 1. MLTut — “NumPy Broadcasting & Vectorization Explained (With Simple Examples)”

A focused explanation of NumPy vectorization and broadcasting. It demonstrates element-wise operations, operations between arrays of different shapes, and why vectorized expressions can replace many explicit Python loops.

**Recommended use:** Watch before the broadcasting and vectorization portion of the hands-on tutorial.

[Watch on YouTube](https://www.youtube.com/watch?v=eMMznK65KOg)

---

## 2. Visual Kernel — “SVD Visualized, Singular Value Decomposition Explained”

A visual treatment of SVD emphasizing singular vectors, singular values, and the geometry of the decomposition.

**Recommended use:** Watch before or after the SVD exercises. It is particularly useful for connecting the factorization formula to geometric intuition.

[Watch on YouTube](https://www.youtube.com/watch?v=vSczTbgc8Rc)

---

## 3. Nick Space Cowboy — “QR Decomposition — Linear Algebra”

A detailed treatment of QR decomposition, including NumPy computation, Gram–Schmidt, Householder transformations, and Givens rotations.

**Recommended use:** The opening sections and NumPy demonstration are sufficient for STA 556; the later derivations are useful optional extensions.

[Watch on YouTube](https://www.youtube.com/watch?v=kyG8YMIfNA0)

---

# 43. Week 9 takeaway

> **Efficient statistical computing often begins by recognizing the linear-algebra structure of a problem and expressing it directly with arrays.**

```text
Python values
      ↓
NumPy arrays
      ↓
Shape
      ↓
Vectorization
      ↓
Broadcasting
      ↓
Matrix operations
      ↓
QR / SVD
      ↓
Stable statistical computation
```

Next week we will use NumPy's numerical machinery for **random-number generation, inverse-transform sampling, and Monte Carlo simulation**.
