---
title: 'Using the good old Fortran L-BFGS-B code'
slug: "using-fortran-lbfgsb"
date: 2023-05-01
categories:
- Programming
- Statistics
tags:
- L-BFGS-B
- Fortran
- C++
- optimization
- programming
- numerical
---

This post is a sequel to the [previous article](/2023/04/using-fortran-lbfgs/)
on how to use the old Fortran code to solve optimization
problems in C++ applications.
This time we consider the L-BFGS-B algorithm for solving
smooth and box-constrained optimization problems
of the form
$$
\begin{align*}
\min_{x}\\, & f(x)\\\\
\text{subject to}\\, & l\le x\le u,
\end{align*}
$$
where $l$ and $u$ are simple bounds for $x\in\mathbb{R}^n$, and can take $-\infty$ and $+\infty$ values.

L-BFGS-B has a widely-used
[Fortran implementation](https://users.iems.northwestern.edu/~nocedal/lbfgsb.html), and is also available in the
[LBFGS++](https://lbfgspp.statr.me/) library. Again, I occasionally
need to compare LBFGS++ with the classical implementation, so this
post gives a workflow of using the Fortran L-BFGS-B in C++.

First, download the source code from
[this link](https://users.iems.northwestern.edu/~nocedal/Software/Lbfgsb.3.0.tar.gz),
and extract the following files: `blas.f`, `lbfgsb.f`, `linpack.f`, and `timer.f`.

The core function in the `lbfgsb.f` file that we are going to use
is called `setulb`. However, one issue here is that the `setulb`
function has arguments of the type `character*60`, which are not
very portable for data exchange between Fortran and C++.
Moreover, `setulb` contains some file operations, which are likely
to cause errors when called inside C++.

Therefore, I recommand using a modified version of `lbfgsb.f`
contained in the
[`lbfgsb3c` R package](https://CRAN.R-project.org/package=lbfgsb3c).
This version removes printing and file operations in the original
version, and uses integer variables to encode strings.
**Note that these two versions are NOT compatible to each other.**

To use the modified version, simply download the
[source package](https://cran.r-project.org/src/contrib/lbfgsb3c_2020-3.2.tar.gz), extract the `lbfgsb.f` file, and replace
the original version. Then the Fortran files can be compiled
into binary code using the commands below:

```bash
gfortran -O2 -c blas.f -o blas.o
gfortran -O2 -c lbfgsb.f -o lbfgsb.o
gfortran -O2 -c linpack.f -o linpack.o
gfortran -O2 -c timer.f -o timer.o
```

On the C++ side, the declaration for the core Fortran function is:


```cpp
// Fortran L-BFGS-B function
extern "C" {

void setulb_(
    const int* n, const int* m,
    double* x, const double* l, const double* u, const int* nbd,
    double* f, double* g,
    const double* factr, const double* pgtol,
    double* wa, int* iwa,
    int* itask, const int* iprint,
    int* icsave, int* lsave, int* isave, double* dsave
);

}
```

Similar to the [previous post](/2023/04/using-fortran-lbfgs/), we
define the Rosenbrock function class, and now we attempt to optimize
the function with each component of $x$ restricted to the range
$2\le x_i \le 4$. Below shows the complete C++ code.

```cpp
#include <iostream>
#include <Eigen/Core>

using Vector = Eigen::VectorXd;
using IntVector = Eigen::VectorXi;

// Fortran L-BFGS-B function
extern "C" {

void setulb_(
    const int* n, const int* m,
    double* x, const double* l, const double* u, const int* nbd,
    double* f, double* g,
    const double* factr, const double* pgtol,
    double* wa, int* iwa,
    int* itask, const int* iprint,
    int* icsave, int* lsave, int* isave, double* dsave
);

}

// Problem class
class Rosenbrock
{
private:
    int n;
public:
    Rosenbrock(int n_) : n(n_) {}
    double operator()(const Vector& x, Vector& grad)
    {
        double fx = 0.0;
        for(int i = 0; i < n; i += 2)
        {
            double t1 = 1.0 - x[i];
            double t2 = 10 * (x[i + 1] - x[i] * x[i]);
            grad[i + 1] = 20 * t2;
            grad[i]     = -2.0 * (x[i] * grad[i + 1] + t1);
            fx += t1 * t1 + t2 * t2;
        }
        return fx;
    }
};

int main()
{
    // Set up problem size
    const int n = 10;
    // Create problem
    Rosenbrock fun(n);
    // Bounds for variables
    Vector lb = Vector::Constant(n, 2.0);
    Vector ub = Vector::Constant(n, 4.0);
    // Initial guess
    Vector x = Vector::Constant(n, 3.0);
    // Space to store gradient
    Vector grad = Vector::Zero(n);
    // Maximum number of iterations
    const int maxiter = 1000;

    // Set up solver parameters
    // Number of correction vectors in L-BFGS-B
    const int m = 10;
    // Bound type indicators
    IntVector nbd(n);
    const double inf = std::numeric_limits<double>::infinity();
    for(int i = 0; i < n; i++)
    {
        if(lb[i] == -inf)
        {
            nbd[i] = (ub[i] == inf) ? 0 : 3;
        } else {
            nbd[i] = (ub[i] == inf) ? 1 : 2;
        }
    }
    // Convergence tolerance
    const double factr = 1e7;
    const double pgtol = 1e-5;
    // Printing options
    // Do not print
    const int iprint = -1;
    // Working space and flags
    Vector wa(2 * m * n + 11 * m * m + 5 * n + 8 * m);
    IntVector iwa(3 * n);
    int itask;
    int icsave;
    int lsave[4];
    int isave[44];
    double dsave[29];

    // Optimization process
    double objval;
    itask = 2;
    int i = 0;
    while (i < maxiter)
    {
        // Call L-BFGS-B routine
        setulb_(&n, &m, x.data(), lb.data(), ub.data(), nbd.data(),
            &objval, grad.data(), &factr, &pgtol,
            wa.data(), iwa.data(), &itask, &iprint,
            &icsave, lsave, isave, dsave);

        std::cout << "i = " << i << ", itask = " << itask << std::endl;
        if (itask == 4 || itask == 20 || itask == 21)
        {
            // Compute objective function value and gradient
            objval = fun(x, grad);
            std::cout << "   x      = " <<  x.transpose() << std::endl;
            std::cout << "   grad   = " <<  grad.transpose() << std::endl;
            std::cout << "   objval = " <<  objval << std::endl;
        } else if (itask >= 6 && itask <= 8) {
            // Converged
            break;
        } else if (itask == 1) {
            // New x, update iteration number
            i = isave[29];
        } else {
            // Errors
            std::cout << "itask = " << itask << std::endl;
            throw std::runtime_error("abnormal exit");
        }
    }

    // Print final results
    std::cout << "================================================" << std::endl;
    std::cout << "x    = " << x.transpose() << std::endl;
    std::cout << "grad = " << grad.transpose() << std::endl;
    std::cout << "# iterations        = " << i + 1 << std::endl;
    std::cout << "# function calls    = " << isave[33] << std::endl;
    std::cout << "Final f             = " << objval << std::endl;
    std::cout << "Final ||proj_grad|| = " << dsave[12] << std::endl;

    return 0;
}
```

This program (assuming the file name is `lbfgsb_example.cpp`)
can be compiled as follows, where `/path/to/Eigen` is replaced
by the path to the Eigen header files:

```bash
g++ -std=c++11 -O2 -I/path/to/Eigen lbfgsb_example.cpp *.o -o lbfgsb_example.out -lgfortran -lquadmath
```

Running the program `./lbfgs_example.out` shows the following output:

```
i = 0, itask = 21
   x      = 3 3 3 3 3 3 3 3 3 3
   grad   =  7204 -1200  7204 -1200  7204 -1200  7204 -1200  7204 -1200
   objval = 18020
i = 0, itask = 20
   x      = 2.68377 3.31623 2.68377 3.31623 2.68377 3.31623 2.68377 3.31623 2.68377 3.31623
   grad   =  4175.46 -777.281  4175.46 -777.281  4175.46 -777.281  4175.46 -777.281  4175.46 -777.281
   objval = 7566.25
i = 0, itask = 1
i = 1, itask = 20
   x      = 2 4 2 4 2 4 2 4 2 4
   grad   = 2 0 2 0 2 0 2 0 2 0
   objval = 5
i = 1, itask = 1
i = 2, itask = 7
================================================
x    = 2 4 2 4 2 4 2 4 2 4
grad = 2 0 2 0 2 0 2 0 2 0
# iterations        = 3
# function calls    = 3
Final f             = 5
Final ||proj_grad|| = 0
```
