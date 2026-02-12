---
title : Computational Statistics (Fall 2025)
comments: true
---

### Overview

This course attempts to introduce useful methods, algorithms, and tools for statistical
computing, and will also discuss some recent advances in computational statistics
related to optimization and sampling. It will roughly be divided into three parts:

- Numerical linear algebra
- Optimization
- Simulation and sampling

Frankly speaking, every method we are going to introduce can be a big research topic
in itself (e.g. eigenvalue computation, optimization algorithms), so we do not attempt to
cover every method in full details. Instead, the goal of this course is to assemble a toolbox
for **accurate**, **efficient**, and **reliable** statistical computing.

### Computing vs Programming

Programming is one critical part of computing, but it is not all. Writing better code with the
same algorithm may bring ~2x speedup of your program, but a better algorithm design may give you
10~100x or even more.

This course does not teach you programming. It teaches you how to better solve problems and write faster and more reliable code.

### Programming Language

There is no preference on the programming language to use, but I will primarily use R to
illustrate the implementation of algorithms, and will use C++ for the computationally intensive
parts. From my own experience, unless you need to rely on GPU-based
libraries such as deep learning frameworks, the combination of R and C++ can solve >90% of
common computing problems in statistical and machine learning models.

### Course Slides

#### Part I: Numerical Linear Algebra

- [Lecture 1](/teaching/compstat/lec1.html): Introduction to statistical computing
  and computational statistics.
- [Lecture 2](/teaching/compstat/lec2.html): Triangular and tridiagonal linear systems,
  LU decomposition, Cholesky decomposition.
- [Lecture 3](/teaching/compstat/lec3.html): Case studies of spline fitting and regression,
  LDL' decomposition, QR decomposition.
- [Lecture 4](/teaching/compstat/lec4.html): Iterative methods, conjugate gradient method,
  eigenvalue computation.
- [Lecture 5](/teaching/compstat/lec5.html): Singular value decomposition, sparse matrix,
  case studies of statistical models on sparse data.

#### Part II: Optimization

- [Lecture 6](/teaching/compstat/lec6.html): Convex sets and convex functions,
  convex optimization problems, (projected) gradient descent method and its convergence properties.
- [Lecture 7](/teaching/compstat/lec7.html): Nonsmooth optimization: subgradient and subdifferential, subgradient methods, proximal gradient descent.
- [Lecture 8](/teaching/compstat/lec8.html): Advanced nonsmooth optimization via operator splitting methods: Douglas-Rachford splitting algorithm, Davis-Yin splitting algorithm, proximal-proximal-gradient method.
- [Lecture 9](/teaching/compstat/lec9.html): Alternating direction method of multipliers, coordinate descent algorithm, case studies.

#### Part III: Simulation and Sampling

- [Lecture 10](/teaching/compstat/lec10.html): Basic and classical simulation problems, inverse transform algorithm, rejection sampling.
- [Lecture 11](/teaching/compstat/lec11.html): Importance sampling, measure transport sampling.
- [Lecture 12](/teaching/compstat/lec12.html): Markov chain Monte Carlo, Metropolis-Hastings algorithm, Gibbs sampler.
- [Lecture 13](/teaching/compstat/lec13.html): Langevin algorithm, Hamiltonian Monte Carlo.
