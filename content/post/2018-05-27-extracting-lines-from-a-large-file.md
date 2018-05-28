---
title: "Extracting Specific Lines from a Large (Compressed) Text File"
slug: "extracting-lines-from-a-large-file"
date: 2018-05-27
categories:
- R
- Programming
tags:
- r
- sed
- text-file
- dataset
- data-science
- big-data
- data-tidying
---

A few days ago a friend asked me the following question: how to efficiently
extract some specific lines from a large text file, possibily compressed by Gzip?
He mentioned that he tried some R functions such as `read.table(skip = ...)`,
but found that reading the data was too slow. Hence he was looking for some
alternative ways to extracting the data.

This is a common task in preprocessing large data sets, since in data exploration,
very often we want to peek at a small subset of the whole data to gain some insights.
If the data are stored in a text file, then we want to extract some specific lines
with the given line numbers. After I got one solution, I felt this might be useful for
future reference, so below are some of my notes for this problem.

After a quick search on [StackOverflow](https://stackoverflow.com/a/83347),
it turns out that the solution is to let the
right tool do the right thing. Assuming a UNIX-like environment, the `sed` command
is the way to go: to extract line 5 to line 8 from file `somefile.txt`, simply run

```bash
sed -n '5,8p' somefile.txt
```

This is very straightforward if you want to read consecutive lines, but things are
more complicated here:

1. We need to extract potentially discontiguous lines, for example, the line numbers
are saved in an R vector.
2. The text file is compressed by Gzip, and we do not want to extract the whole file.

The solution to the first point is still using `sed`: for example, to extract
line 2, 4, and 6, the following command works.

```bash
sed -n '2p;4p;6p' somefile.txt
```

Even better, we can actually generate this command and run it within
R, so that we do not need to manually type the command when the list is long. We will
demonstrate this at the end.

For the second issue, we can solve it by utilizing the powerful pipe mechanism in
UNIX-like systems. In short, we uncompress the file using `zcat`, and send the
streamed data to `sed` for extraction. For example, if the text file is compressed as
`somefile.tar.gz`, then the following command reads line 2, 4, and 6 of the original file:

```
zcat somefile.tar.gz | sed -n '2p;4p;6p;7q'
```

Note that we add a `7q` parameter at the end, which asks `sed` to exit after reading line 7.
This is **very important** for reading large data sets, since otherwise `sed` will walk
through the whole data till the end.

Combining things together, we can write a simple R function to accomplish this task
under different scenarios: input file can either be in plain text or be compressed,
results can be saved to a new file or be returned as a character vector, etc.

```r
#' Extract lines from a large text file
#'
#' @param infile    path to the input file
#' @param lines     a vector of line numbers
#' @param outfile   if `NULL`, return the result as a vector of character strings;
#'                  otherwise the path to the output file
#' @param gzip      whether the input file is compressed using Gzip
extract_lines = function(infile, lines, outfile = NULL, gzip = FALSE)
{
    # e.g., lines = c(1, 20, 100)
    lines = as.integer(lines)

    # 1p;20p;100p;101q
    sed_arg = paste(lines, "p", sep = "", collapse = ";")
    sed_arg = paste(sed_arg, ";", max(lines) + 1, "q", sep = "")

    # sed -n '1p;20p;100p;101q'
    sed_command = sprintf("sed -n '%s'", sed_arg)

    # If outfile is not `NULL`, redirect the result to a file
    out_command = if(is.null(outfile)) "" else sprintf("> %s", as.character(outfile))

    # If file is compressed, combine `zcat` with `sed`
    if(gzip)
        command = sprintf("zcat %s | %s %s", infile, sed_command, out_command)
    else
        command = sprintf("%s %s %s", sed_command, infile, out_command)

    # Execute command
    system(command, intern = is.null(outfile))
}
```
