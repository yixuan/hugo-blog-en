---
title: "How to run regression on large datasets in R"
date: 2011-10-02
categories:
- Programming
- R
- Statistics
tags:
- database
- dataset
- matrix
- memory
- R
- regression
- RSQLite
- SQL
- SQLite
---

It's well known that R is a memory based software, meaning that datasets must be copied into memory before
being manipulated. For small or medium scale datasets, this doesn't cause any troubles.
However, when you need to deal with larger ones, for instance, financial time series or log data
from the Internet, the consumption of memory is always a nuisance.

Just to give a simple illustration, you can put in the following code into R to allocate a matrix
named x and a vector named y.

```r
set.seed(123);
n = 5000000;
p = 5;
x = matrix(rnorm(n * p), n, p);
x = cbind(1, x);
bet = c(2, rep(1, p));
y = c(x %*% bet) + rnorm(n);
```

If I try to run a regression on x and y with the built-in function `lm()`, I get the error.

```r
> lm(y ~ 0 + x);
Error: cannot allocate vector of size 19.1 Mb
In addition: Warning messages:
1: In lm.fit(x, y, offset = offset, singular.ok = singular.ok, ...) :
  Reached total allocation of 1956Mb: see help(memory.size)
2: In lm.fit(x, y, offset = offset, singular.ok = singular.ok, ...) :
  Reached total allocation of 1956Mb: see help(memory.size)
3: In lm.fit(x, y, offset = offset, singular.ok = singular.ok, ...) :
  Reached total allocation of 1956Mb: see help(memory.size)
4: In lm.fit(x, y, offset = offset, singular.ok = singular.ok, ...) :
  Reached total allocation of 1956Mb: see help(memory.size)
```

The parameters of my machine are:

- CPU: Intel Core i5-2410M @ 2.30 GHz
- Memory: 2GB
- OS: Windows 7 64-bit
- R: 2.13.1 32-bit

In R, each numeric number occupies `8 Bytes`, so we can estimate that x and y will only occupy
`5000000 * 7 * 8 / 1024 ^ 2 Bytes = 267 MB`, far less than the total memory size of 2GB.
However, the memory is still used up since `lm()` will compute many variables apart from
x and y, for example, the fitted values and residuals.

If we are only interested in the coefficient estimation, we can directly use matrix operations
to compute beta hat:
```r
beta.hat = solve(t(x) %*% x, t(x) %*% y);
```

This runs successfully on my machine and the process is very fast of only about 0.6 seconds
(I use an optimized Rblas.dll, [download here](http://yixuan.cos.name/en/wp-content/uploads/2011/10/Rblas_gotoblas.tar.gz)). Nevertheless, if the sample size is larger,
the matrix operation may also be unavailable. To provide an estimation,
when the sample size is as large as `2GB / 7 / 8 Bytes = 38347922`,
x and y themselves will swallow all the memory, let alone other temporary variables created
in the computation.

So how can we cope with this problem?

One approach to avoid too much consumption of memory is to use a database system and excute SQL
statements on it. Database restores data on the hard disk and uses a small buffer to run SQL,
so you don't need to worry about the memory;
it's just a matter of how long it takes to accomplish the computation.

R supports many database systems among which [SQLite](http://www.sqlite.org/) is the lightest and the most convenient.
There is an **RSQLite** package in R that allows you to read/write data from/to an **SQLite** database,
as well as executing SQL statements on it and fetching results back to R. Therefore, if we can
"translate" our algorithm into SQL statements, then the size of data we can deal with will only depend on
the hard disk size and the execution time we can tolerate.

To continue with the example above, I'll illustrate how to complete the regression using database and SQL.
First, we shall write the data into a database file on the hard disk. The code is

```r
gc();
dat = as.data.frame(x);
rm(x);
gc();
dat$y = y;
rm(y);
gc();
colnames(dat) = c(paste("x", 0:p, sep = ""), "y");
gc();

# Will also load the DBI package
library(RSQLite);
# Using the SQLite database driver
m = dbDriver("SQLite");
# The name of the database file
dbfile = "regression.db";
# Create a connection to the database
con = dbConnect(m, dbname = dbfile);
# Write the data in R into database
if(dbExistsTable(con, "regdata")) dbRemoveTable(con, "regdata");
dbWriteTable(con, "regdata", dat, row.names = FALSE);
# Close the connection
dbDisconnect(con);
# Garbage collection
rm(dat);
gc();
```

I use a lot of `rm()` and `gc()` functions to remove unused temporary variables and cleanse the memory.
When all is done, you'll find a regression.db file in your working directory whose size is about 320M.
Then it comes the most important step -- translate the regression algorithm into SQL.

Recall that the expression of beta hat can be written in a quite simple form:

$$\hat{\beta}=(X'X)^{-1}X'y$$

Also note that however large the sample size $n$ is, $X'X$ and $X'y$ are always of the size
$(p+1) \times (p+1)$. If the number of variables is not very large, the inverse and multiplication
of matrices of that size could be easily handled by R, so our main target is to compute
$X'X$ and $X'y$ in SQL.

Rewrite $X$ as $X=(\mathbf{x\_0,x\_1,\ldots,x_p})$, then $X'X$ could be expressed as

$$\left(\begin{array}{cccc}\mathbf{x_{0}'x_{0}} & \mathbf{x_{0}'x_{1}} & \ldots & \mathbf{x_{0}'x_{p}}\\\mathbf{x_{1}'x_{0}} & \mathbf{x_{1}'x_{1}} & \ldots & \mathbf{x_{1}'x_{p}}\\\vdots & \vdots & \ddots & \vdots\\\mathbf{x_{p}'x_{0}} & \mathbf{x_{p}'x_{1}} & \ldots & \mathbf{x_{p}'x_{p}}\end{array}\right)$$

And each element in the matrix can be calculated using SQL, for example,

```sql
select sum(x0 * x0), sum(x0 * x1) from regdata;
```

We can then use R to generate the character strings of SQL statement and send it to SQLite.
The code is as follows:

```r
m = dbDriver("SQLite");
dbfile = "regression.db";
con = dbConnect(m, dbname = dbfile);
# Get variable names
vars = dbListFields(con, "regdata");
xnames = vars[-length(vars)];
yname = vars[length(vars)];
# Generate SQL statements to compute X'X
mult = outer(xnames, xnames, paste, sep = "*");
lower.index = lower.tri(mult, TRUE);
mult.lower = mult[lower.index];
sql = paste("sum(", mult.lower, ")", sep = "", collapse = ",");
sql = sprintf("select %s from regdata", sql);
txx.lower = unlist(dbGetQuery(con, sql), use.names = FALSE);
txx = matrix(0, p + 1, p + 1);
txx[lower.index] = txx.lower;
txx = t(txx);
txx[lower.index] = txx.lower;
# Generate SQL statements to compute X'Y
sql = paste(xnames, yname, sep = "*");
sql = paste("sum(", sql, ")", sep = "", collapse = ",");
sql = sprintf("select %s from regdata", sql);
txy = unlist(dbGetQuery(con, sql), use.names = FALSE);
txy = matrix(txy, p + 1);
# Compute beta hat in R
beta.hat.DB = solve(txx, txy);
t6 = Sys.time();
```

We can check whether the results are the same by calculating the maximum absolute difference:

```r
> max(abs(beta.hat - beta.hat.DB));
[1] 3.028688e-13
```

A difference of rounding errors.

The computation takes about 17 seconds, far more than the matrix operation, but it consumes nearly
no extra memory to accomplish the computation, which is a typical example of "trade time for space".
Furthermore, you may have noticed that the computation of
`sum(x0*x0), sum(x0*x1), ..., sum(x5*x5)` can be parallelized by opening several connections to the database
simultaneously, so if you have a multi-processor server, you may drastically reduce the time
after some arrangement.

The whole source code could be [downloaded here](https://github.com/downloads/yixuan/en/DB_regression.tar.gz).
