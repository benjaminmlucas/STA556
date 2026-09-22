# 6.1 Activity

## Root Finding, Newton--Raphson, Gradient Descent & Statistical Optimization

**Tools:** Python, NumPy, SciPy, matplotlib, Jupyter/VS Code

## Learning objectives

By the end of this tutorial, you should be able to:

-   formulate a root-finding problem;
-   implement bisection;
-   implement Newton--Raphson;
-   diagnose convergence failures;
-   define objective functions;
-   compute analytical and numerical derivatives;
-   implement gradient descent;
-   explore learning-rate behavior;
-   fit OLS via optimization;
-   formulate a negative log-likelihood;
-   estimate parameters using SciPy;
-   validate custom implementations.

------------------------------------------------------------------------

# Part 0 --- Set up

Create:

``` text
notebooks/week12_optimization.ipynb
```

Import:

``` python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

from scipy.optimize import (
    root_scalar,
    minimize
)
```

Create:

``` python
rng = np.random.default_rng(556)
```

------------------------------------------------------------------------

# Part 1 --- Define a root problem

``` python
def f(x):
    return x ** 2 - 2
```

Evaluate:

``` python
f(1)
f(2)
```

### Questions

1.  What signs do the values have?
2.  What does this imply?
3.  What is the exact positive solution?

------------------------------------------------------------------------

# Part 2 --- Visualize the root

``` python
x_grid = np.linspace(
    0,
    3,
    300
)

plt.plot(
    x_grid,
    f(x_grid)
)

plt.axhline(0)
plt.xlabel("x")
plt.ylabel("f(x)")
plt.show()
```

Identify the root visually.

------------------------------------------------------------------------

# Part 3 --- One bisection step

Start:

``` python
a = 1.0
b = 2.0

midpoint = (
    a + b
) / 2
```

Inspect:

``` python
f(a)
f(midpoint)
f(b)
```

Which half contains the root?

------------------------------------------------------------------------

# Part 4 --- Implement bisection

``` python
def bisection(
    f,
    a,
    b,
    tol=1e-8,
    max_iter=100
):
    if f(a) * f(b) > 0:
        raise ValueError(
            "Root is not bracketed."
        )

    for iteration in range(
        max_iter
    ):
        midpoint = (
            a + b
        ) / 2

        value = f(midpoint)

        if abs(value) < tol:
            return (
                midpoint,
                iteration + 1
            )

        if f(a) * value < 0:
            b = midpoint
        else:
            a = midpoint

    return midpoint, max_iter
```

Compare your result with:

``` python
np.sqrt(2)
```

------------------------------------------------------------------------

# Part 5 --- Invalid brackets

Try:

``` python
bisection(
    f,
    2,
    3
)
```

Why is failing immediately useful?

------------------------------------------------------------------------

# Part 6 --- Validate bisection with SciPy

``` python
result = root_scalar(
    f,
    bracket=[1, 2],
    method="bisect"
)
```

Inspect:

``` python
result.root
result.converged
result.iterations
```

------------------------------------------------------------------------

# Part 7 --- Newton derivative

Define:

``` python
def f_prime(x):
    return 2 * x
```

Start at:

``` python
x = 1.5
```

Perform one Newton update manually.

------------------------------------------------------------------------

# Part 8 --- Implement Newton--Raphson

``` python
def newton(
    f,
    f_prime,
    x0,
    tol=1e-8,
    max_iter=100
):
    x = x0
    history = [x]

    for iteration in range(
        max_iter
    ):
        derivative = f_prime(x)

        if abs(derivative) < 1e-12:
            raise ValueError(
                "Derivative is too close to zero."
            )

        x_new = (
            x
            - f(x)
            / derivative
        )

        history.append(
            x_new
        )

        if abs(x_new - x) < tol:
            return (
                x_new,
                iteration + 1,
                history
            )

        x = x_new

    return x, max_iter, history
```

Run from `x0=1.5`.

------------------------------------------------------------------------

# Part 9 --- Visualize Newton convergence

Plot the sequence of iterates against `sqrt(2)`.

Compare the number of iterations with bisection.

------------------------------------------------------------------------

# Part 10 --- Validate Newton with SciPy

``` python
result = root_scalar(
    f,
    x0=1.5,
    fprime=f_prime,
    method="newton"
)
```

Inspect root, iteration count, and convergence.

------------------------------------------------------------------------

# Part 11 --- Poor starting values

Define:

``` python
def cubic(x):
    return (
        x ** 3
        - 2 * x
        + 2
    )
```

and:

``` python
def cubic_prime(x):
    return (
        3 * x ** 2
        - 2
    )
```

Try multiple starting values.

Record:

``` text
x0
root/result
iterations
converged?
```

------------------------------------------------------------------------

# Part 12 --- Numerical derivatives

``` python
def numerical_derivative(
    f,
    x,
    h=1e-5
):
    return (
        f(x + h)
        - f(x - h)
    ) / (
        2 * h
    )
```

Compare the exact and numerical derivative at `x=1.5`.

------------------------------------------------------------------------

# Part 13 --- Finite-difference step size

Try:

``` text
h = 1e-1
1e-3
1e-5
1e-8
1e-12
```

Calculate absolute error.

Why is smaller not always better?

------------------------------------------------------------------------

# Part 14 --- Simple optimization

Define:

``` python
def objective(x):
    return (
        x - 3
    ) ** 2
```

and:

``` python
def gradient(x):
    return (
        2
        * (
            x - 3
        )
    )
```

Plot the objective.

------------------------------------------------------------------------

# Part 15 --- One gradient-descent update

Start:

``` python
x = 8.0
learning_rate = 0.1
```

Calculate one update manually.

Did it move toward the minimum?

------------------------------------------------------------------------

# Part 16 --- Implement gradient descent

``` python
def gradient_descent(
    objective,
    gradient,
    x0,
    learning_rate=0.1,
    tol=1e-8,
    max_iter=1000
):
    x = x0
    history = [
        (
            x,
            objective(x)
        )
    ]

    for iteration in range(
        max_iter
    ):
        x_new = (
            x
            - learning_rate
            * gradient(x)
        )

        history.append(
            (
                x_new,
                objective(x_new)
            )
        )

        if abs(x_new - x) < tol:
            return (
                x_new,
                iteration + 1,
                history
            )

        x = x_new

    return x, max_iter, history
```

------------------------------------------------------------------------

# Part 17 --- Learning-rate experiment

Run with:

``` text
0.01
0.1
0.5
1.1
```

Record:

``` text
learning rate
iterations
final x
final loss
```

Plot loss vs. iteration.

------------------------------------------------------------------------

# Part 18 --- Multivariable gradient descent

Define:

``` python
def objective_2d(
    theta
):
    return (
        (
            theta[0] - 2
        ) ** 2
        + 2
        * (
            theta[1] + 1
        ) ** 2
    )
```

Gradient:

``` python
def gradient_2d(
    theta
):
    return np.array([
        2 * (
            theta[0] - 2
        ),
        4 * (
            theta[1] + 1
        )
    ])
```

Start at:

``` python
theta = np.array([
    8.0,
    5.0
])
```

Implement vector gradient descent.

Expected minimum:

``` text
[2, -1]
```

------------------------------------------------------------------------

# Part 19 --- Simulate regression data

``` python
n = 200

x = rng.normal(
    size=n
)

X = np.column_stack([
    np.ones(n),
    x
])

beta_true = np.array([
    5.0,
    2.0
])

y = (
    X @ beta_true
    + rng.normal(
        0,
        1.5,
        size=n
    )
)
```

------------------------------------------------------------------------

# Part 20 --- MSE loss

``` python
def mse_loss(
    beta,
    X,
    y
):
    residuals = (
        y - X @ beta
    )

    return np.mean(
        residuals ** 2
    )
```

Evaluate at zero coefficients and at `beta_true`.

------------------------------------------------------------------------

# Part 21 --- MSE gradient

``` python
def mse_gradient(
    beta,
    X,
    y
):
    n = len(y)

    return (
        2 / n
        * X.T
        @ (
            X @ beta - y
        )
    )
```

Inspect the gradient shape.

------------------------------------------------------------------------

# Part 22 --- Numerical gradient checker

Write:

``` python
def numerical_gradient(
    objective,
    theta,
    h=1e-5
):
    grad = np.zeros_like(
        theta,
        dtype=float
    )

    for j in range(
        len(theta)
    ):
        theta_plus = theta.copy()
        theta_minus = theta.copy()

        theta_plus[j] += h
        theta_minus[j] -= h

        grad[j] = (
            objective(theta_plus)
            - objective(theta_minus)
        ) / (
            2 * h
        )

    return grad
```

Compare the numerical and analytical gradients with `np.allclose()`.

------------------------------------------------------------------------

# Part 23 --- Fit OLS with gradient descent

Start:

``` python
beta = np.zeros(
    X.shape[1]
)
```

Iterate using your gradient.

Store loss history.

Stop when parameter changes become small.

------------------------------------------------------------------------

# Part 24 --- Compare with least squares

``` python
beta_lstsq, *_ = (
    np.linalg.lstsq(
        X,
        y,
        rcond=None
    )
)
```

Compare:

``` python
beta
beta_lstsq
beta_true
```

Explain why the first two should agree.

------------------------------------------------------------------------

# Part 25 --- Visualize regression convergence

Plot:

``` text
iteration
vs.
MSE
```

What does convergence look like?

------------------------------------------------------------------------

# Part 26 --- Feature scaling

Add a predictor on a much larger scale:

``` python
x2 = rng.normal(
    0,
    1000,
    size=n
)
```

Try gradient descent before and after standardizing predictors.

Compare convergence.

------------------------------------------------------------------------

# Part 27 --- Normal likelihood

Simulate:

``` python
data = rng.normal(
    loc=7,
    scale=2,
    size=200
)
```

Assume `sigma=2` is known.

Define:

``` python
def normal_nll_mu(
    mu,
    data,
    sigma
):
    residuals = (
        data - mu
    )

    return (
        len(data)
        * np.log(sigma)
        + np.sum(
            residuals ** 2
        ) / (
            2 * sigma ** 2
        )
    )
```

------------------------------------------------------------------------

# Part 28 --- Plot the negative log-likelihood

Evaluate the NLL over a grid of `mu` values.

Plot the curve.

Where does the minimum occur relative to the sample mean?

------------------------------------------------------------------------

# Part 29 --- Estimate the mean with SciPy

``` python
result = minimize(
    lambda theta: normal_nll_mu(
        theta[0],
        data,
        sigma=2
    ),
    x0=np.array([
        0.0
    ])
)
```

Inspect:

``` python
result.x
result.fun
result.success
result.message
result.nit
```

Compare with:

``` python
data.mean()
```

------------------------------------------------------------------------

# Part 30 --- Estimate mean and SD jointly

Parameterize:

``` text
theta = [mu, log_sigma]
```

Write:

``` python
def normal_nll(
    theta,
    data
):
    mu = theta[0]

    sigma = np.exp(
        theta[1]
    )

    residuals = (
        data - mu
    )

    return (
        len(data)
        * np.log(sigma)
        + np.sum(
            residuals ** 2
        ) / (
            2 * sigma ** 2
        )
    )
```

------------------------------------------------------------------------

# Part 31 --- Optimize both parameters

``` python
result = minimize(
    normal_nll,
    x0=np.array([
        0.0,
        0.0
    ]),
    args=(
        data,
    )
)
```

Recover:

``` python
mu_hat = result.x[0]

sigma_hat = np.exp(
    result.x[1]
)
```

Compare with sample summaries.

------------------------------------------------------------------------

# Part 32 --- Why log SD?

Explain why optimizing unrestricted `sigma` can be problematic.

How does:

``` text
sigma = exp(log_sigma)
```

enforce the correct parameter domain?

------------------------------------------------------------------------

# Part 33 --- Starting-value sensitivity

Repeat the two-parameter optimization from:

``` text
[0, 0]
[10, 0]
[-20, 2]
[100, -2]
```

Record success, iterations, estimates, and final objective values.

------------------------------------------------------------------------

# Part 34 --- Nonconvex objective

Define:

``` python
def nonconvex(x):
    return (
        np.sin(
            3 * x
        )
        + 0.1
        * x ** 2
    )
```

Plot the function.

Run `minimize()` from several starting points.

Do all runs find the same minimum?

------------------------------------------------------------------------

# Part 35 --- Root finding or optimization?

Choose the best framing.

### A

``` text
x³ - 5x + 1 = 0
```

### B

Estimate regression coefficients by minimizing squared error.

### C

Solve a likelihood score equation.

### D

Estimate parameters by maximizing likelihood.

### E

Find where two nonlinear curves intersect.

Explain each answer.

------------------------------------------------------------------------

# Part 36 --- Mini optimization study

Simulate:

``` python
n = 300

x1 = rng.normal(
    size=n
)

x2 = rng.normal(
    size=n
)

X = np.column_stack([
    np.ones(n),
    x1,
    x2
])

beta_true = np.array([
    3.0,
    1.5,
    -2.0
])

y = (
    X @ beta_true
    + rng.normal(
        0,
        2,
        size=n
    )
)
```

Complete:

1.  Define MSE loss.
2.  Derive/code its gradient.
3.  Numerically check the gradient.
4.  Fit coefficients with gradient descent.
5.  Plot loss over iterations.
6.  Compare with `np.linalg.lstsq()`.
7.  Fit using `scipy.optimize.minimize()`.
8.  Compare all estimates.
9.  Record convergence diagnostics.
10. Explain which approach you would use in production.

------------------------------------------------------------------------

# Part 37 --- Optimization audit

Before trusting an optimization result, answer:

``` text
What objective is being optimized?
Are we minimizing or maximizing?
Are derivatives correct?
Are parameters constrained?
What are the starting values?
What is the stopping rule?
Did the algorithm converge?
Is the result sensitive to initialization?
Can it be checked against an analytical/library result?
```

------------------------------------------------------------------------

# Part 38 --- Git checkpoint

``` bash
git status
git add .
git commit -m "Complete Week 12 numerical optimization exercises"
git push
```

------------------------------------------------------------------------

# Part 39 --- Final reflection

### 1. Root finding

What does it mean to find a root?

### 2. Bisection

Why is it robust?

### 3. Newton--Raphson

Where does the update come from?

### 4. Starting values

Why do they matter?

### 5. Gradient

What information does it contain?

### 6. Gradient descent

What role does the learning rate play?

### 7. OLS

How is regression an optimization problem?

### 8. MLE

Why minimize the negative log-likelihood?

### 9. Constraints

Why optimize `log(sigma)` instead of `sigma`?

### 10. Validation

What should be checked before trusting optimizer output?

------------------------------------------------------------------------

# Completion checklist

-   [ ] Created Week 12 notebook
-   [ ] Defined and visualized a root problem
-   [ ] Implemented bisection
-   [ ] Validated bisection with SciPy
-   [ ] Implemented Newton--Raphson
-   [ ] Visualized Newton convergence
-   [ ] Validated Newton with SciPy
-   [ ] Explored poor starting values
-   [ ] Implemented numerical differentiation
-   [ ] Investigated finite-difference step size
-   [ ] Defined an objective
-   [ ] Implemented gradient descent
-   [ ] Compared learning rates
-   [ ] Implemented multivariable gradient descent
-   [ ] Simulated regression data
-   [ ] Defined MSE loss
-   [ ] Derived the MSE gradient
-   [ ] Numerically checked the gradient
-   [ ] Fit OLS with gradient descent
-   [ ] Compared with `np.linalg.lstsq`
-   [ ] Investigated feature scaling
-   [ ] Defined a negative log-likelihood
-   [ ] Estimated a mean using `minimize`
-   [ ] Estimated mean and SD jointly
-   [ ] Used a log transformation for a positive parameter
-   [ ] Explored starting-value sensitivity
-   [ ] Explored a nonconvex objective
-   [ ] Checked SciPy convergence diagnostics
-   [ ] Completed the root-vs-optimization challenge
-   [ ] Completed the mini optimization study
-   [ ] Audited an optimization workflow
-   [ ] Committed Week 12 work to Git
-   [ ] Pushed work to GitHub

------------------------------------------------------------------------

# What you should now understand

``` text
Statistical problem
      ↓
Equation / loss / likelihood
      ↓
Root or optimum
      ↓
Derivative information
      ↓
Numerical iteration
      ↓
Convergence diagnostics
      ↓
Parameter estimate
      ↓
Validation
```

Next week we will focus on **debugging, testing, defensive programming,
and exception handling**.
