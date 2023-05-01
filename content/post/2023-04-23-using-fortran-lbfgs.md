---
title: 'Using the good old Fortran L-BFGS code'
slug: "using-fortran-lbfgs"
date: 2023-04-23
categories:
- Programming
- Statistics
tags:
- L-BFGS
- Fortran
- C++
- optimization
- programming
- numerical
---

L-BFGS is a well-known and widely-used optimization algorithm for smooth 
and unconstrained optimization problems. It was
[originally implemented in Fortran](https://users.iems.northwestern.edu/~nocedal/lbfgs.html),
and also has some more recent implementations including
[libLBFGS](https://www.chokkan.org/software/liblbfgs/) and my own
[LBFGS++](https://lbfgspp.statr.me/).

The Fortran code was written more than 30 years ago, and looks a bit exotic from
today's perspective. However, it is still one of the most stable and mature
implementations of the L-BFGS algorithm, and is typically used as a baseline
in testing and benchmarking.

This post briefly introduces how to use the Fortran L-BFGS code in a typical C++
program, which is at least useful to myself as I occasionally need to compre LBFGS++
with the classical implementation.

To do this, the first step is to download the source code from
[this link](https://users.iems.northwestern.edu/~nocedal/Software/lbfgs_um.tar.gz),
and extract the `lbfgs.f` file from the zipped package. We can then compile the code
as below: 

```bash
gfortran -O2 -c lbfgs.f -o lbfgs.o
```

This will generate a binary file `lbfgs.o` that contains the compiled Fortran function.

Then we start writing the C++ program. As usual, we first include some necessary header
files and the declaration of the Fortran function:

```cpp
#include <iostream>
#include <Eigen/Core>

using Vector = Eigen::VectorXd;

// Fortran L-BFGS function
extern "C" {

void lbfgs_(
    const int* n, const int* m,
    double* x, const double* f, const double* g,
    const int* diagco, const double* diag,
    const int* iprint, const double* eps, const double* xtol,
    double* w, int* iflag
);

}
```

Note that the included [Eigen](https://eigen.tuxfamily.org/index.php?title=Main_Page)
library is not necessary for simple problems, but I find it very useful in
numerical and statistical computing, so I implement the objective function later using
classes defined in the Eigen library.

Also notice that the function we declared above is called `lbfgs_`, which has an
underscore appended (the Fortran function in `lbfgs.f` is called `LBFGS`).
This is a convention of the mixed programming in Fortran and C++.

Next, we define a C++ class to represent the optimization problem we want to solve:

```cpp
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
```

This is not the only method to define an optimization problem, but is the one
I recommend in using LBFGS++. In fact, this piece of code is directly taken from
the [README](https://github.com/yixuan/LBFGSpp/blob/master/README.md) of LBFGS++.
The key idea is to construct an object `fun` such that `fun(x, grad)` returns
the objective function value evaluated at a vector `x`, and overwrites the `grad`
vector with the gradient at `x`.

Finally, below shows a complete example that calls the Fortran L-BFGS to minimize
the Rosenbrock function.

```cpp
#include <iostream>
#include <Eigen/Core>

using Vector = Eigen::VectorXd;

// Fortran L-BFGS function
extern "C" {

void lbfgs_(
    const int* n, const int* m,
    double* x, const double* f, const double* g,
    const int* diagco, const double* diag,
    const int* iprint, const double* eps, const double* xtol,
    double* w, int* iflag
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
    // Initial guess
    Vector x = Vector::Zero(n);
    // Space to store gradient
    Vector grad = Vector::Zero(n);
    // Maximum number of iterations
    const int maxiter = 1000;

    // Set up solver parameters
    // Number of correction vectors in L-BFGS
    const int m = 10;
    // Do not provide H0
    const int diagco = 0;
    Vector diag(n);
    // Convergence tolerance
    const double eps = 1e-6;
    // Machine precision
    const double xtol = std::numeric_limits<double>::epsilon();
    // Printing options
    int iprint[2];
    iprint[0] = 1;  // Print in every iteration
    iprint[1] = 0;  // 0-3, larger value for more output

    // Working space and flags
    Vector work(n * (2 * m + 1) + 2 * m);
    int iflag = 0;

    // Optimization process
    double objval;
    int i;
    for (i = 0; i < maxiter; i++)
    {
        // Compute objective function value and gradient
        objval = fun(x, grad);
        // Call L-BFGS routine
        lbfgs_(&n, &m, x.data(), &objval, grad.data(),
            &diagco, diag.data(), iprint, &eps, &xtol,
            work.data(), &iflag);
        // If iflag = 1, then continue iteration
        if (iflag == 1)
        {
            continue;
        } // If iflag = 0, then the solver finishes
        else if (iflag == 0) {
            break;
        } // If iflag < 0, then some error occurs
        else
        {
            std::cout << "L-BFGS solver failed with code " << iflag << std::endl;
            return iflag;
        }
    }

    // Print final results
    std::cout << "================================================" << std::endl;
    std::cout << "x    = " << x.transpose() << std::endl;
    std::cout << "grad = " << grad.transpose() << std::endl;
    std::cout << "# function calls = " << i + 1 << std::endl;
    std::cout << "Final f          = " << objval << std::endl;
    std::cout << "Final ||grad||   = " << grad.norm() << std::endl;

    return 0;
}
```

Some explanations of the code above:

1. The key logic of the `lbfgs_()` function is to return a flag variable `iflag`, which
instructs the user what to do next.
2. If `iflag == 1`, then the user needs to compute the objective function value `objval`
and the gradient `grad` at the current `x` vector, and supply these objects to the next
call of `lbfgs_()`.
3. If `iflag == 0`, then the algorithm converges successfully, and the `x` vector contains
the solution.
4. If `iflag < 0`, then some error occurs, and the value of `iflag` indicates the type of
error.
5. The meaning of each parameter and error code can be found in the `lbfgs.f` file.

We can compile the C++ code (assuming the file name is `lbfgs_example.cpp`) and link it
with the Fortran binary file as follows, where `/path/to/Eigen` is replaced by the path
to the Eigen header files (for example, if it is in the current directory, then just
use `-I.`):

```bash
g++ -std=c++11 -O2 -I/path/to/Eigen lbfgs_example.cpp lbfgs.o -o lbfgs_example.out -lgfortran -lquadmath
```

Running the program `./lbfgs_example.out` shows the following output:

```
*************************************************
  N=   10   NUMBER OF CORRECTIONS=10
       INITIAL VALUES
 F=  5.000D+00   GNORM=  4.472D+00
*************************************************

   I   NFN    FUNC        GNORM       STEPLENGTH

   1    3     4.003D+00   6.463D+00   5.752D-02
   2    5     3.468D+00   1.089D+01   2.164D-01
   3    6     2.556D+00   1.234D+01   1.000D+00
   4    7     1.988D+00   1.172D+01   1.000D+00
   5    9     1.264D+00   4.103D+00   5.422D-01
   6   11     1.187D+00   9.217D+00   2.953D-01
   7   12     1.023D+00   9.277D+00   1.000D+00
   8   13     6.057D-01   5.417D+00   1.000D+00
   9   14     3.956D-01   6.327D+00   1.000D+00
  10   15     2.095D-01   7.052D+00   1.000D+00
  11   17     9.589D-02   9.211D-01   1.774D-01
  12   19     7.521D-02   4.677D+00   3.936D-01
  13   20     5.602D-02   4.549D+00   1.000D+00
  14   21     1.515D-02   1.327D+00   1.000D+00
  15   22     5.411D-03   2.175D+00   1.000D+00
  16   23     6.644D-04   7.377D-01   1.000D+00
  17   24     8.160D-05   3.797D-01   1.000D+00
  18   25     2.421D-06   1.425D-03   1.000D+00
  19   26     1.602D-07   7.079D-03   1.000D+00
  20   27     5.210D-10   1.006D-03   1.000D+00
  21   28     2.043D-12   6.054D-05   1.000D+00
  22   29     2.358D-16   1.408D-07   1.000D+00

 THE MINIMIZATION TERMINATED WITHOUT DETECTING ERRORS.
 IFLAG = 0
================================================
x    = 1 1 1 1 1 1 1 1 1 1
grad = 5.87724e-08 -2.2613e-08 5.87724e-08 -2.2613e-08 5.87724e-08 -2.2613e-08 5.87724e-08 -2.2613e-08 5.87724e-08 -2.2613e-08
# function calls = 29
Final f          = 2.35773e-16
Final ||grad||   = 1.40811e-07
```
