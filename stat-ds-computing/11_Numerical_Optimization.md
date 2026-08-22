# 6.1 Numerical Optimization

## Why this week matters

Many statistical procedures can be written as optimization problems.

Examples include ordinary least squares, maximum likelihood estimation,
logistic regression, regularized regression, neural-network training,
nonlinear curve fitting, and numerical equation solving.

The central question for Week 12 is:

> **How do we turn a statistical estimation problem into an objective
> function that a numerical algorithm can solve?**

This week focuses on root finding, bisection, Newton--Raphson, objective
and loss functions, gradients and Hessians, gradient descent, learning
rates, convergence criteria, ordinary least squares as optimization,
maximum likelihood as optimization, and validation against SciPy.

------------------------------------------------------------------------

# 1. Root finding vs. optimization

Two common numerical tasks are:

``` text
Root finding:
find x such that f(x) = 0

Optimization:
find x* that minimizes f(x)
```

These are related because a differentiable optimum often satisfies:

``` text
f'(x*) = 0
```

------------------------------------------------------------------------

# 2. A root-finding example

Suppose:

``` text
f(x) = x² - 2
```

We want:

``` text
x² - 2 = 0
```

Define:

``` python
def f(x):
    return x ** 2 - 2
```

The positive solution is `sqrt(2)`, but we will approximate it
numerically.

------------------------------------------------------------------------

# 3. Bracketing

If `f(a)` and `f(b)` have opposite signs and `f` is continuous, at least
one root lies between `a` and `b`.

For example:

``` python
f(1)
f(2)
```

give opposite signs.

This provides the bracket:

``` text
[1, 2]
```

------------------------------------------------------------------------

# 4. Bisection

Bisection repeatedly halves a valid bracket.

``` text
[a,b]
  ↓
m = (a+b)/2
  ↓
keep half containing sign change
  ↓
repeat
```

A simple implementation:

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

    for _ in range(max_iter):
        midpoint = (a + b) / 2
        value = f(midpoint)

        if abs(value) < tol:
            return midpoint

        if f(a) * value < 0:
            b = midpoint
        else:
            a = midpoint

    return midpoint
```

Bisection is slow but robust when its assumptions hold.

------------------------------------------------------------------------

# 5. Newton--Raphson

Newton's method uses derivative information:

``` text
x_(k+1)
=
x_k - f(x_k)/f'(x_k)
```

Geometrically, the tangent line at the current point predicts where the
function crosses zero.

------------------------------------------------------------------------

# 6. Deriving Newton's method

Use a first-order Taylor approximation:

``` text
f(x)
≈
f(x_k) + f'(x_k)(x-x_k)
```

Set this equal to zero and solve for `x`:

``` text
x
=
x_k - f(x_k)/f'(x_k)
```

This becomes the next iterate.

------------------------------------------------------------------------

# 7. Newton--Raphson in Python

``` python
def newton(
    f,
    f_prime,
    x0,
    tol=1e-8,
    max_iter=100
):
    x = x0

    for _ in range(max_iter):
        derivative = f_prime(x)

        if abs(derivative) < 1e-12:
            raise ValueError(
                "Derivative is too close to zero."
            )

        x_new = (
            x
            - f(x) / derivative
        )

        if abs(x_new - x) < tol:
            return x_new

        x = x_new

    return x
```

------------------------------------------------------------------------

# 8. Why Newton can be fast

Near a well-behaved simple root, Newton's method can converge
approximately quadratically.

But it may fail because of poor starting values, near-zero derivatives,
oscillation, divergence, multiple roots, or invalid parameter regions.

> **Fast convergence does not imply guaranteed convergence.**

------------------------------------------------------------------------

# 9. Convergence criteria

Possible stopping rules include:

``` text
|x_(k+1) - x_k| < tolerance
```

and:

``` text
|f(x_k)| < tolerance
```

A robust implementation should also have a maximum number of iterations.

------------------------------------------------------------------------

# 10. Validate with SciPy

SciPy provides:

``` python
from scipy.optimize import root_scalar
```

For bisection:

``` python
result = root_scalar(
    f,
    bracket=[1, 2],
    method="bisect"
)
```

For Newton:

``` python
result = root_scalar(
    f,
    x0=1.5,
    fprime=f_prime,
    method="newton"
)
```

Inspect:

``` python
result.root
result.converged
result.iterations
```

------------------------------------------------------------------------

# 11. Optimization objectives

An optimization problem has the form:

``` text
minimize f(theta)
```

The function may be called:

``` text
objective
loss
cost
negative log-likelihood
residual sum of squares
```

The optimizer searches parameter space for values that reduce the
objective.

------------------------------------------------------------------------

# 12. Gradient

For:

``` text
f(theta_1,...,theta_p)
```

the gradient is:

``` text
∇f(theta)
=
[
∂f/∂theta_1,
...,
∂f/∂theta_p
]^T
```

The gradient points in the direction of steepest local increase.

Therefore `-∇f` points toward local decrease.

------------------------------------------------------------------------

# 13. Gradient descent

The update is:

``` text
theta_(k+1)
=
theta_k - alpha ∇f(theta_k)
```

where `alpha` is the learning rate.

Conceptually:

``` text
current parameters
      ↓
calculate gradient
      ↓
move opposite gradient
      ↓
new parameters
      ↓
repeat
```

------------------------------------------------------------------------

# 14. Learning rate

If the learning rate is too small:

``` text
convergence is slow
```

If it is too large:

``` text
overshooting
oscillation
divergence
```

The step size is part of the numerical method.

------------------------------------------------------------------------

# 15. Hessian

For a multivariable objective, the Hessian is the matrix of second
derivatives.

It describes local curvature and appears in Newton optimization and
uncertainty approximations.

For multiple dimensions, Newton's method can be written:

``` text
theta_(k+1)
=
theta_k - H(theta_k)^(-1) ∇f(theta_k)
```

In practice, solve the linear system instead of explicitly forming an
inverse.

------------------------------------------------------------------------

# 16. OLS as optimization

Linear regression:

``` text
y = X beta + epsilon
```

Ordinary least squares minimizes:

``` text
RSS(beta)
=
||y - X beta||²
```

Example loss:

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

------------------------------------------------------------------------

# 17. Gradient of OLS loss

For:

``` text
L(beta)
=
(1/n)||y-Xbeta||²
```

the gradient is:

``` text
∇L(beta)
=
(2/n)X^T(Xbeta-y)
```

In NumPy:

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

------------------------------------------------------------------------

# 18. Gradient descent for regression

Initialize:

``` python
beta = np.zeros(
    X.shape[1]
)
```

Then repeatedly update:

``` python
beta = (
    beta
    - learning_rate
    * mse_gradient(
        beta,
        X,
        y
    )
)
```

Compare the result with:

``` python
np.linalg.lstsq(
    X,
    y,
    rcond=None
)
```

Two correct computational routes should agree closely.

------------------------------------------------------------------------

# 19. Feature scaling

Gradient descent can behave poorly when predictor scales differ
dramatically.

Standardization:

``` python
means = X.mean(axis=0)
sds = X.std(axis=0)

Z = (
    X - means
) / sds
```

can make the optimization geometry much easier.

------------------------------------------------------------------------

# 20. Maximum likelihood estimation

For density:

``` text
f(x | theta)
```

the likelihood is:

``` text
L(theta)
=
product f(x_i | theta)
```

The log-likelihood is:

``` text
ell(theta)
=
sum log f(x_i | theta)
```

Maximum likelihood chooses:

``` text
theta_hat
=
argmax ell(theta)
```

------------------------------------------------------------------------

# 21. Negative log-likelihood

Most optimization libraries minimize.

Therefore:

``` text
maximize log-likelihood
```

is equivalent to:

``` text
minimize negative log-likelihood
```

This is one of the central patterns in statistical computing.

------------------------------------------------------------------------

# 22. Normal mean MLE

Suppose:

``` text
X_i ~ Normal(mu, sigma²)
```

with known `sigma`.

Ignoring constants, minimizing the negative log-likelihood in `mu` is
equivalent to minimizing:

``` text
sum (x_i - mu)²
```

The numerical optimizer should recover approximately the sample mean.

------------------------------------------------------------------------

# 23. Constrained parameters

If `sigma` is unknown, it must satisfy:

``` text
sigma > 0
```

One useful parameterization is:

``` text
eta = log(sigma)
```

so:

``` text
sigma = exp(eta)
```

The optimizer can search over any real `eta` while always producing a
valid positive `sigma`.

------------------------------------------------------------------------

# 24. SciPy `minimize`

Typical use:

``` python
from scipy.optimize import minimize

result = minimize(
    objective,
    x0=initial_parameters
)
```

Always inspect:

``` python
result.x
result.fun
result.success
result.message
result.nit
```

Do not trust `result.x` without checking convergence.

------------------------------------------------------------------------

# 25. Local vs. global minima

A nonconvex objective can have several local minima.

Different initial values may lead to different solutions.

Possible strategies include multiple starting values, better
initialization, visualization, or global optimization methods.

------------------------------------------------------------------------

# 26. Convexity

For a convex objective:

``` text
every local minimum is a global minimum
```

OLS loss is convex.

This makes least-squares optimization easier to reason about than many
complex machine-learning objectives.

------------------------------------------------------------------------

# 27. Numerical differentiation

If an analytical derivative is inconvenient, approximate it:

``` text
f'(x)
≈
[f(x+h)-f(x-h)]/(2h)
```

Implementation:

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

------------------------------------------------------------------------

# 28. Finite-difference step size

If `h` is too large, approximation error can be substantial.

If `h` is too small, floating-point cancellation can dominate.

This illustrates a common theme in numerical computing:

> **More precision in an input setting does not always produce more
> accurate computation.**

------------------------------------------------------------------------

# 29. Gradient checking

Analytical gradients should be checked against numerical approximations.

This is extremely useful when implementing statistical or
machine-learning algorithms from scratch.

A wrong gradient may still produce code that runs.

------------------------------------------------------------------------

# 30. Optimization diagnostics

Always inspect:

``` text
objective history
gradient magnitude
parameter changes
iteration count
convergence flag
final objective
sensitivity to starting values
```

An optimizer returning numbers does not prove it solved the intended
problem.

------------------------------------------------------------------------

# 31. Common mistakes

-   maximizing when the library minimizes;
-   incorrect derivative signs;
-   learning rate too large;
-   no maximum iteration limit;
-   ignored parameter constraints;
-   ignored convergence messages;
-   explicit matrix inverses when a solve is preferable;
-   failure to check sensitivity to starting values.

------------------------------------------------------------------------

# 32. Statistical optimization workflow

``` text
Statistical model
      ↓
Define equation / objective
      ↓
Derive or approximate derivatives
      ↓
Choose starting values
      ↓
Choose numerical method
      ↓
Iterate
      ↓
Check convergence
      ↓
Validate
      ↓
Interpret
```

------------------------------------------------------------------------

# 33. Key ideas

By the end of Week 12, you should be able to explain:

1.  Root finding vs. optimization.
2.  Bracketing and bisection.
3.  Newton--Raphson updates.
4.  Why Newton's method can converge quickly or fail.
5.  Why starting values matter.
6.  Objective and loss functions.
7.  Gradients and Hessians.
8.  Gradient descent.
9.  Learning-rate behavior.
10. OLS as optimization.
11. The OLS gradient.
12. Why feature scaling matters.
13. MLE as optimization.
14. Negative log-likelihood.
15. Constrained parameter transformations.
16. Local vs. global minima.
17. Numerical differentiation.
18. Gradient checking.
19. Convergence diagnostics.
20. Validation against trusted routines.

------------------------------------------------------------------------

# 34. Recommended reading

## SciPy --- Optimization and Root Finding

https://docs.scipy.org/doc/scipy/reference/optimize.html

## SciPy --- `root_scalar`

https://docs.scipy.org/doc/scipy/reference/generated/scipy.optimize.root_scalar.html

## SciPy --- Optimization Tutorial

https://docs.scipy.org/doc/scipy/tutorial/optimize.html

## NumPy --- Linear Algebra

https://numpy.org/doc/stable/reference/routines.linalg.html

------------------------------------------------------------------------

# 35. YouTube recommendations

## 1. MathAndPhysics --- Newton-Raphson Algorithm in Python

A focused implementation of Newton--Raphson in Python, including the
update rule and practical considerations.

**Recommended use:** Watch before implementing Newton's method from
scratch.

[Watch on YouTube](https://www.youtube.com/watch?v=T3q0hZjXG3g)

------------------------------------------------------------------------

## 2. Dataquest --- Gradient Descent From Scratch In Python

A practical implementation of gradient descent for linear regression. It
covers loss, partial derivatives, parameter updates, learning rates, and
the training loop.

**Recommended use:** Watch alongside the OLS and gradient-descent
sections.

[Watch on YouTube](https://www.youtube.com/watch?v=-cs5D91eBLE)

------------------------------------------------------------------------

## 3. StatQuest --- Gradient Descent

A visual explanation of gradients, step sizes, and movement across a
loss surface.

**Recommended use:** Optional conceptual reinforcement before the
multivariable exercises.

[Find on
YouTube](https://www.youtube.com/results?search_query=StatQuest+Gradient+Descent)

------------------------------------------------------------------------

# 36. Week 12 takeaway

> **Statistical estimation often becomes a numerical problem once we
> express the model as an equation or objective function.**

``` text
Statistical model
      ↓
Equation / loss / likelihood
      ↓
Root or optimum
      ↓
Derivative information
      ↓
Numerical iterations
      ↓
Convergence
      ↓
Parameter estimate
      ↓
Validation
```

Next week we move into **debugging, testing, defensive programming, and
exception handling**.
