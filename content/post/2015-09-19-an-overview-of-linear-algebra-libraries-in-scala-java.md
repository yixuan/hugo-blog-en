---
title: "An overview of linear algebra libraries in Scala/Java"
date: 2015-09-19
categories:
- Programming
- Statistics
tags:
- matrix
- linear algebra
- statistics
- data science
- programming
- big data
- Scala
- Java
---

This semester I'm taking a course in big data computing using Scala/Spark, and
we are asked to finish a course project related to big data analysis. Since
statistical modeling heavily relies on linear algebra, I investigated some
existing libraries in Scala/Java that deal with matrix and linear algebra
algorithms.

# 1. Set-up

Scala/Java libraries are usually distributed as `*.jar` files. To use them in Scala,
we can create a directory to hold them and set up the environment variable to let
Scala know about this path. For example, we first create a folder named `scala_lib`
in home directory, and then edit the `.bash_profile` file
(create one if it does not exist), adding the following line:

```bash
export CLASSPATH=$CLASSPATH:~/scala_lib/*
```

To make it effective for the current session, type in the terminal

```bash
source .bash_profile
```

Then the `.jar` files can be downloaded to this directory and Scala will recognize it.



# 2. Common operations

Most of the libraries discussed here support basic matrix operations, such as
creating a matrix, getting and setting elements or sub-matrices, matrix multiplications,
solving linear equations, etc.

For each library discussed below, we use it to accomplish the following tasks, in
order to demonstrate the usage of the library:

1. Create a 3 by 6 matrix $A$
2. Fill the matrix with random numbers
3. Set $A\_{1,1}=A\_{3,6}$
4. Get the 3 by 3 sub-matrix $B=A\_{1:3,1:3}$
5. Set the sub-matrix $A\_{1:3,2:4}=B$
6. Calculate the matrix product $C=A^\prime B$
7. Solve linear equation $Bx=a$, where $a$ is the first column of $A$

# 3. JAMA

## Introduction

[JAMA](http://math.nist.gov/javanumerics/jama/) is a basic linear algebra package
for Java which provides a matrix class and a number of matrix decomposition classes.
The matrix class supports basic operations such as matrix addition, multiplication,
transpose, norm calculation etc.

## Installation

Simply download the `.jar` file in the Java class path.

```bash
cd ~/scala_lib
wget http://math.nist.gov/javanumerics/jama/Jama-1.0.3.jar
```

## Usage Example

In Scala console,

```scala
// Import library
import Jama._

// Create the matrix
val A = new Matrix(3, 6)

// Fill the matrix with random numbers
val r = new scala.util.Random(0)

for(i <- 0 until A.getRowDimension())
    for(j <- 0 until A.getColumnDimension())
        A.set(i, j, r.nextDouble())

// JAMA does not provide methods to print the matrix,
// but we can view the data using the following trick
A.getArray().foreach(row => println(row.mkString("\t")))

// Set the first value to be the last value
A.set(0, 0, A.get(A.getRowDimension()-1, A.getColumnDimension()-1))

// Get a sub-matrix, 1st row to 3rd row, 1st column to 3rd column
val B = A.getMatrix(0, 2, 0, 2)

// Set a sub-matrix of A to B, 1st row to 3rd row,
// 2nd column to 4th column
A.setMatrix(0, 2, 1, 3, B)

// Matrix product C=A'B
val C = A.transpose().times(B)

// Solve linear equation
val a = A.getMatrix(0, 2, 0, 0)
val x = B.solve(a)
```

## Documention

The full documentation is at [http://math.nist.gov/javanumerics/jama/doc/](http://math.nist.gov/javanumerics/jama/doc/).



# 4. Apache Commons Math

## Introduction

[Apache Commons Math](http://commons.apache.org/proper/commons-math/index.html)
is an Apache project aiming to address the most common mathematical and statistical
problems that are not available in the standard Java language. It supports both
dense and sparse matrix classes, equipped with basic operations as well as matrix
decomposition algorithms.

## Installation

Download a zip file and extract the `.jar` file into the Java class path.

```bash
cd ~/scala_lib
wget http://supergsego.com/apache//commons/math/binaries/commons-math3-3.5-bin.tar.gz
tar xzf commons-math3-3.5-bin.tar.gz commons-math3-3.5/commons-math3-3.5.jar
mv commons-math3-3.5/commons-math3-3.5.jar .
rm commons-math3-3.5-bin.tar.gz
rm -r commons-math3-3.5
```

## Usage Example

In Scala console,

```scala
// Import library
import org.apache.commons.math3.linear._

// Create the matrix
val A = new Array2DRowRealMatrix(3, 6)

// Fill the matrix with random numbers
val r = new scala.util.Random(0)

for(i <- 0 until A.getRowDimension())
    for(j <- 0 until A.getColumnDimension())
        A.setEntry(i, j, r.nextDouble())

// Set the first value to be the last value
A.setEntry(0, 0,
    A.getEntry(A.getRowDimension() - 1, A.getColumnDimension() - 1))

// Get a sub-matrix, 1st row to 3rd row, 1st column to 3rd column
val B = A.getSubMatrix(0, 2, 0, 2)

// Set a sub-matrix of A to B, 1st row to 3rd row,
// 2nd column to 4th column
A.setSubMatrix(B.getData(), 0, 1)

// Matrix product C=A'B
val C = A.transpose().multiply(B)

// Solve linear equation
val solver = new LUDecomposition(B).getSolver()
val a = A.getColumnVector(0)
val x = solver.solve(a)
```

## Documention

The full documentation is at
[http://commons.apache.org/proper/commons-math/userguide/linear.html](http://commons.apache.org/proper/commons-math/userguide/linear.html).



# 5. la4j

## Introduction

[la4j](http://la4j.org/) is a light weight linear algebra library for Java,
supporting both dense and sparse matrices, as well as matrix decomposition
algorithms.

## Installation

Download the `.jar` file into the Java class path.

```bash
cd ~/scala_lib
wget http://central.maven.org/maven2/org/la4j/la4j/0.5.5/la4j-0.5.5.jar
```

## Usage Example

In Scala console,

```scala
// Import library
import org.la4j.matrix._
import org.la4j.linear._

// Create the matrix
val A = DenseMatrix.zero(3, 6)

// Fill the matrix with random numbers
val r = new scala.util.Random(0)

for(i <- 0 until A.rows())
    for(j <- 0 until A.columns())
        A.set(i, j, r.nextDouble())

// Set the first value to be the last value
A.set(0, 0, A.get(A.rows() - 1, A.columns() - 1))

// Get a sub-matrix, 1st row to 3rd row, 1st column to 3rd column
val B = A.slice(0, 0, 3, 3)

// Set a sub-matrix of A to B, 1st row to 3rd row,
// 2nd column to 4th column.
// It seems that there is no direct sub-matrix setting function,
// so we set columns one by one
for(i <- 0 to 2)
    A.setColumn(i + 1, B.getColumn(i))

// Matrix product C=A'B
val C = A.transpose().multiply(B)

// Solve linear equation
val solver = new GaussianSolver(B)
val a = A.getColumn(0)
val x = solver.solve(a)
```

## Documention

The full documentation is at [http://la4j.org/apidocs/](http://la4j.org/apidocs/).



# 6. EJML

## Introduction

Efficient Java Matrix Library ([EJML](http://ejml.org/wiki/index.php))
is a linear algebra library for manipulating dense matrices.
It is designed to be computationally efficient.

## Installation

Download a zip file and extract the `.jar` file into the Java class path.

```bash
cd ~/scala_lib
wget http://downloads.sourceforge.net/project/ejml/v0.28/ejml-v0.28-libs.zip
unzip ejml-v0.28-libs.zip
mv ejml-v0.28-libs/* .
rm ejml-v0.28-libs.zip
rm -r ejml-v0.28-libs
```

## Usage Example

In Scala console,

```scala
// Import library
import org.ejml.simple._

// Create the matrix
val A = new SimpleMatrix(3, 6)

// Fill the matrix with random numbers
val r = new scala.util.Random(0)

for(i <- 0 until A.numRows())
    for(j <- 0 until A.numCols())
        A.set(i, j, r.nextDouble())

// Set the first value to be the last value
A.set(0, 0, A.get(A.numRows() - 1, A.numCols() - 1))

// Get a sub-matrix, 1st row to 3rd row, 1st column to 3rd column
val B = A.extractMatrix(0, 3, 0, 3)

// Set a sub-matrix of A to B, 1st row to 3rd row,
// 2nd column to 4th column
// It seems that there is no direct sub-matrix setting function,
// so we set element by element
for(i <- 0 to 2)
    for(j <- 0 to 2)
        A.set(i, j + 1, B.get(i, j))

// Matrix product C=A'B
val C = A.transpose().mult(B)

// Solve linear equation
val a = A.extractVector(false, 0)
val x = B.solve(a)
```

## Documention

The full documentation is at [http://ejml.org/javadoc/](http://ejml.org/javadoc/).



# 7. Breeze

## Introduction

[Breeze](https://github.com/scalanlp/breeze) is a Scala libary for
machine learning and numerical computing. It contains matrix and vector classes
and many other components such as statistical distributions, optimization,
integration, etc. Since this library is written in Scala, the syntax is generally
more elegant and convenient than those implemented in pure Java.

## Installation

At the time of writing there is no officially provided `.jar` file on the web, so it is
suggested to build the library from source code, which may take some efforts.

The following commands automatically downloads the source code of Breeze and
a necessary building tool [sbt](http://www.scala-sbt.org/), and then builds the
`.jar` file from source.

```bash
cd ~/scala_lib
wget https://github.com/scalanlp/breeze/archive/master.zip
unzip master.zip
cd breeze-master
wget https://dl.bintray.com/sbt/native-packages/sbt/0.13.9/sbt-0.13.9.tgz
tar xzf sbt-0.13.9.tgz
./sbt/bin/sbt assembly
mv target/scala-2.*/breeze-*.jar ..
cd ..
rm master.zip
rm -r breeze-master
```

## Usage Example

In Scala console,

```scala
// Import library
import breeze.linalg._
import breeze.numerics._

// Create the matrix
val A = DenseMatrix.zeros[Double](3, 6)

// Fill the matrix with random numbers
val r = new scala.util.Random(0)

for(i <- 0 until A.rows)
    for(j <- 0 until A.cols)
        A(i, j) = r.nextDouble()

// Set the first value to be the last value
A(0, 0) = A(A.rows - 1, A.cols - 1)

// Get a sub-matrix, 1st row to 3rd row, 1st column to 3rd column
// We need to make a copy here since without it changing B will
// also change A
val B = A( :: , 0 to 2).copy

// Set a sub-matrix of A to B, 1st row to 3rd row,
// 2nd column to 4th column
A( :: , 1 to 3) := B

// Matrix product C=A'B
val C = A.t * B

// Solve linear equation
val a = A( :: , 0)
val x = B \ a
```

## Documention

* [Quick Start](https://github.com/scalanlp/breeze/wiki/Quickstart)
* [Linear Algebra Cheat Sheet](https://github.com/scalanlp/breeze/wiki/Linear-Algebra-Cheat-Sheet)

# 8. Summary

All the libraries written in Java have similar syntax and function names. There
is also [a benchmark](http://lessthanoptimal.github.io/Java-Matrix-Benchmark/runtime/2013_10_Corei7v2600/)
of matrix operations on different libraries, including the ones mentioned here.

Breeze takes advantage of the Scala syntax, which makes the code more elegant
and easier to write. For example matrix elements can be accessed or set using
parentheses, and operator overloading allows users to write matrix operations
just like in mathematical formulas. Due to this reason Breeze seems to be a
good choice for matrix manipulation in Scala.
