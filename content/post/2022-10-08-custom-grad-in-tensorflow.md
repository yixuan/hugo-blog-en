---
title: 'Custom gradient in Tensorflow'
slug: "custom-grad-in-tensorflow"
date: 2022-10-08
categories:
- Programming
- Statistics
tags:
- Tensorflow
- automatic differentiation
- gradient
- matrix calculus
- Jacobian matrix
---

# Motivation

Deep learning frameworks such as PyTorch and Tensorflow provide excellent
auto-differentiation support for matrices and vectors. They have included
many built-in functions and operators that can be combined together to
create complicated yet auto-differentiable functions. However, in some cases
we prefer to manually define the gradient of a function, instead of relying
on automatic differentiation; yet we still allow this function to be embedded
into a larger program, which has end-to-end auto-differentiation support.

The motivation above can be better illustrated using a simple example.
Suppose we are using Tensorflow 2.x, and we have an input vector $x$
of length $n$. The output scalar $y$ is computed as $y=f_3(f_2(f_1(x)))$,
where $f_1:\mathbb{R}^n\rightarrow\mathbb{R}^n$,
$f_2:\mathbb{R}^n\rightarrow\mathbb{R}^n$, and
$f_3:\mathbb{R}^n\rightarrow\mathbb{R}$.

Now suppose that $f_1$ and $f_3$ are built-in functions of Tensorflow,
for example, `tf.math.sin()` and `tf.math.reduce_mean()`, respectively,
but we want to define our own implementation of $f_2$. This may be useful
if $f_2$ is not yet implemented by Tensorflow, or if we have a better
algorithm. For example, suppose $f_2(x)=\Phi(x)$, the standard normal c.d.f.,
and when $x$ is a vector or a matrix, $f_2$ applies to each entry of $x$.
Of course, we can implement $f_2$ using the Tensorflow function
`tf.math.erf()`, but below we use a SciPy implementation for illustration
purpose:

```python
import math
import scipy
import tensorflow as tf

def f2(x):
    xnp = x.numpy()
    res = scipy.stats.norm.cdf(xnp)
    return tf.constant(res, dtype=x.dtype)

x = tf.constant([-1.0, 0.0, 2.0])
print(f2(x))
# tf.Tensor([0.15865526 0.5        0.97724986], shape=(3,), dtype=float32)
```

Now we can compute our output $y$ as:

```python
f1out = tf.math.sin(x)
f2out = f2(f1out)
y = tf.math.reduce_mean(f2out)
print(y)
# tf.Tensor(0.5061485, shape=(), dtype=float32)
```

This looks nice! However, if we want to compute the gradient
$\partial y/\partial x$, we are running into trouble:

```python
with tf.GradientTape() as tape:
    tape.watch(x)
    f1out = tf.math.sin(x)
    f2out = f2(f1out)
    y = tf.math.reduce_mean(f2out)
dydx = tape.gradient(y, x)
print(dydx is None)
# True
```

The gradient $\partial y/\partial x$ should be nonzero, but the program gives
a `None` result for `dydx`. Obviously, this is because our `f2` is not
auto-differentiable in Tensorflow, as it relies on Scipy code that is
outside the Tensorflow computational graph.

# Custom gradient

But theoretically, if we can also provide the derivative
$\partial f_2/\partial x$, then we should be able to compute
$\partial y/\partial x$ using the chain rule. Fortunately,
Tensorflow provides a function decorator `@tf.custom_gradient` exactly to
do this. The solution is also intuitive:
we give `f2` some additional information, the gradient.
We have already known that $\Phi^{\prime}(x)=\phi(x)$,
the normal p.d.f. Then we define `f2` in the following way:

```python
@tf.custom_gradient
def f2(x):
    xnp = x.numpy()
    res = scipy.stats.norm.cdf(xnp)
    res = tf.constant(res, dtype=x.dtype)

    def grad(upstream):
        pdf = scipy.stats.norm.pdf(xnp)
        pdf = tf.constant(pdf, dtype=x.dtype)
        return upstream * pdf

    return res, grad

with tf.GradientTape() as tape:
    tape.watch(x)
    f1out = tf.math.sin(x)
    f2out = f2(f1out)
    y = tf.math.reduce_mean(f2out)
dydx = tape.gradient(y, x)
print(y)
# <tf.Tensor: shape=(), dtype=float32, numpy=0.5061485>
print(dydx)
# <tf.Tensor: shape=(3,), dtype=float32, numpy=array([ 0.05042773,  0.13298076, -0.03660103], dtype=float32)>
```

It works! But... how should we write the `grad()` function, and what does
the parameter `upstream` mean?

# How it works

In fact, the example above is too special that it hides many details.
To fully understand the use of `@tf.custom_gradient`, we shall consider
a more general setting. We still use the example $y=f_3(f_2(f_1(x)))$,
but now
$f_1:\mathbb{R}^n\rightarrow\mathbb{R}^m$,
$f_2:\mathbb{R}^m\rightarrow\mathbb{R}^p$, and
$f_3:\mathbb{R}^p\rightarrow\mathbb{R}$.
For convenience we also let $u=f_1(x)\in\mathbb{R}^m$ and
$v=f_2(u)\in\mathbb{R}^p$.
By the chain rule of derivative, we have

$$\left[\frac{\partial y}{\partial x^{T}}\right]\_{1\times n}=\left[\frac{\partial y}{\partial v^{T}}\right]\_{1\times p}\cdot\left[\frac{\partial v}{\partial u^{T}}\right]\_{p\times m}\cdot\left[\frac{\partial u}{\partial x^{T}}\right]\_{m\times n}$$

In the formula above,
$\partial y/\partial x^T=(\partial y/\partial x_1,\ldots,\partial y/\partial x_n)$
is a **row vector**, and $\partial v/\partial u^T$ is a $p\times m$
**Jacobian matrix**

$$J=\begin{bmatrix}\frac{\partial v_{1}}{\partial u_{1}} & \frac{\partial v_{1}}{\partial u_{2}} & \cdots & \frac{\partial v_{1}}{\partial u_{m}}\\\\
\frac{\partial v_{2}}{\partial u_{1}} & \frac{\partial v_{2}}{\partial u_{2}} & \cdots & \frac{\partial v_{2}}{\partial u_{m}}\\\\
\vdots & \vdots & \ddots & \vdots\\\\
\frac{\partial v_{p}}{\partial u_{1}} & \frac{\partial v_{p}}{\partial u_{2}} & \cdots & \frac{\partial v_{p}}{\partial u_{m}}
\end{bmatrix}.$$

Now if you want to define the `f2` function in Tensorflow with gradient
support, you should complete the following two steps:

1. The **forward pass**, which receives an $m\times 1$ vector $u$ as input,
and returns a $p\times 1$ vector $v$ as output, where $v=f_2(u)$.
2. The **backward pass**, which receives a $p\times 1$ vector
$[\partial y/\partial v^T]^T$ as input, and returns
$J^T[\partial y/\partial v^T]^T$
as output. The `upstream` parameter in the `grad()` function is basically
the vector $[\partial y/\partial v^T]^T$, and your task inside `grad()`
is to compute $J^T[\partial y/\partial v^T]^T$. Note that since $J$ is
$p\times m$ and $[\partial y/\partial v^T]^T$ is $p\times 1$, the result
would be an $m\times 1$ vector, which is exactly
$[\partial y/\partial u^T]^T=J^T[\partial y/\partial v^T]^T$.

Therefore, the `grad()` function essentially does a matrix-vector
multiplication, where the vector is the **upstream gradient**
$[\partial y/\partial v^T]^T$, and the matrix is the **transposed**
Jacobian matrix $J^T$.

But wait... why in our earlier example, we only see an elementwise
multiplication (`upstream * pdf`), but not a matrix-vector one?
In fact, that is why I say the example is special: it can be shown
that the Jacobian matrix there is a **diagonal matrix**, so then
the matrix-vector multiplication reduces to an elementwise
vector-vector multiplication.

# A more general example

Consider another example with a nontrivial Jacobian matrix.
The $f_2$ function is a linear
transformation $f_2(x)=Ax$, where $x\in\mathbb{R}^m$,
$A\in\mathbb{R}^{p\times m}$.
We assume $A$ is a fixed matrix and does not
require gradient. Then we know that the $p\times m$ Jacobian matrix
is $J=\partial f_2/\partial x^T=A$. Suppose $n=m=3$ and $p=5$, and
then the code for this example is:

```python
def f2_generator(A):
    # Define the function with custom gradient
    @tf.custom_gradient
    def f2(x):
        x = tf.reshape(x, (-1, 1))
        res = tf.linalg.matmul(A, x)
        res = tf.squeeze(res)
        # The gradient function
        def grad(upstream):
            upstream = tf.reshape(upstream, (-1, 1))
            g = tf.linalg.matmul(A, upstream, transpose_a=True)
            return tf.squeeze(g)
        # Return the forward result and backward gradient function
        return res, grad
    # Return the actual f2 function
    return f2

tf.random.set_seed(123)
A = tf.random.normal(shape=(5, 3))
with tf.GradientTape() as tape:
    tape.watch(x)
    f1out = tf.math.sin(x)
    f2 = f2_generator(A)
    f2out = f2(f1out)
    y = tf.math.reduce_mean(f2out)
dydx = tape.gradient(y, x)
print(y)
# tf.Tensor(0.4688384, shape=(), dtype=float32)
print(dydx)
# tf.Tensor([-0.1900933  -0.88442993 -0.07907664], shape=(3,), dtype=float32)
```

The overall structure is clear: `f2` computes $Ax$, and `grad` computes $A^Tv$.
But there is something new here. Instead of directly applying
`@tf.custom_gradient` to `f2`, we first define a "generator" `f2_generator`
that will return the actual `f2` function.
This is because we have an additional argument `A`. If we put `A` into `f2`,
then the `grad` function also needs to compute the gradient for `A`.
Hence `f2_generator` is used to receive additional arguments for `f2`, and
`f2` only needs to take the vector `x` as the input.

Of course, a more elegant solution is to let `f2` take two inputs,
`x` and `A`, and compute the gradients for both inputs. The code would be
as follows:

```python
@tf.custom_gradient
def f2(x, A):
    x = tf.reshape(x, (-1, 1))
    res = tf.linalg.matmul(A, x)
    res = tf.squeeze(res)

    def grad(upstream):
        upstream = tf.reshape(upstream, (-1, 1))
        grad_x = tf.linalg.matmul(A, upstream, transpose_a=True)
        grad_A = tf.linalg.matmul(upstream, x, transpose_b=True)
        return tf.squeeze(grad_x), grad_A

    return res, grad

tf.random.set_seed(123)
A = tf.random.normal(shape=(5, 3))
with tf.GradientTape(persistent=True) as tape:
    tape.watch(x)
    tape.watch(A)
    f1out = tf.math.sin(x)
    f2out = f2(f1out, A)
    y = tf.math.reduce_mean(f2out)
dydx = tape.gradient(y, x)
dydA = tape.gradient(y, A)
print(y)
# tf.Tensor(0.4688384, shape=(), dtype=float32)
print(dydx)
# tf.Tensor([-0.1900933  -0.88442993 -0.07907664], shape=(3,), dtype=float32)
print(dydA)
# tf.Tensor(
# [[-0.1682942  0.         0.1818595]
#  [-0.1682942  0.         0.1818595]
#  [-0.1682942  0.         0.1818595]
#  [-0.1682942  0.         0.1818595]
#  [-0.1682942  0.         0.1818595]], shape=(5, 3), dtype=float32)
```

The tricky part here is to compute the gradient for a matrix input.
In general, when you encounter a function $f(x)$ whose input $x$ is
a $p\times m$ matrix, you can first vectorize it into a $pm\times 1$
vector, derive its Jacobian matrix and gradient formula, and then
transform the gradient back to matrix form. The example above directly uses
the fact that $\partial y/\partial A=[\partial y/\partial v^T]^T x^T$
if $v=Ax$, $y=f_3(v)$. For more complicated functions, some knowledge
on matrix calculus would be very helpful, see for example
[Calculus with vectors and matrices](http://paulklein.ca/newsite/teaching/calcvec.pdf)
and
[The Matrix Cookbook](https://www.math.uwaterloo.ca/~hwolkowi/matrixcookbook.pdf).
