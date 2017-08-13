---
title: "Handwriting recognition using R"
slug: "handwriting-recognition-using-r"
date: 2011-12-18
categories:
- Programming
- R
- Statistics
tags:
- correlation
- similarity
- handwriting recognition
- machine learning
- R
---

This title is a bit exaggerating since handwriting recognition is an advanced topic
in machine learning involving complex techniques and algorithms. In this blog I'll
show you a simple demo illustrating how to recognize a single number (0 ~ 9) using R.
The overall process is that, you draw a number in a graphics device in R using your mouse,
and then the program will "guess" what you have input. It is just for **FUN**.

There are two major problems in this number recognition problem, that
is, how to describe the trace of your handwriting, and how to classify
this trace to the give classes (0 ~ 9).

For the first question, we could first detect the motion of your mouse
in the graphics device, and then record the coordinates of you mouse
cursor at a sequence of time points. This could be done via the
`getGraphicsEvent()` function in **grDevices** package. For example, after I
drew a number 2 in the graphics window like below, the coordinates of
each point in the trace were assigned to a pair of variables `px` and `py`.

<p><img src="https://i.imgur.com/257Ng.png" alt="Record trace" class="aligncenter"/></p>

The scatterplot of `px` and `py` versus their orders in the trace is
shown below.

<p><img src="https://i.imgur.com/4gsCV.png" alt="Record points" class="aligncenter"/></p>

To be comparable among different traces, we normalize the Order to be
within (0, 1] (that is, transform 1, 2, ..., n to 1/n, 2/n, ..., 1).
Also, since this recording is discrete but the real trace should be
continuous, we use the `spline()` function to interpolate at unknown
points, resulting in the following figure.

<p><img src="https://i.imgur.com/M0Wos.png" alt="Record splines" class="aligncenter"/></p>

The dots in the figure have normalized orders of 0.02, 0.04,
0.06, ..., 1, at which the x and y coordinates are obtained by
interpolation. Therefore, we could use $r = (x, y)$ where
$x = (x\_1, x\_2, \ldots, x\_{50})^\prime$ and $y = (y\_1, y\_2, \ldots, y\_{50})^\prime$ to
represent the information of the number 2 I have drawn. Somewhat
confused by the operations above? Well, the idea behind this
normalization and interpolation is simple: use 50 "uniformly
ordered" points (I call them "recording points") to represent the trace.

So it comes to the second question -- given a trace, how to classify
it? Obviously we first need a training set, the recording points of
number 0 to number 9 generated as above. Then we'll compare the
given trace with each one in the training set and find out which
number resembles it most.

Several criteria could be used to measure the similarity, but some
important rules should be considered. We still use $r = (x, y)$ to
represent the recording points of a trace, and use $Sim(r\_1, r\_2)$ to
stand for the similarity between two traces. Notice that this
similarity should not be sensitive to the scale and location of
traces. That is, if I draw a number in another location in the
window, or in a larger or smaller size, the recognition should not be
influenced. In mathematics, this could be expressed by

$$Sim(r_1, r_2) = Sim(k_1 r_1 + b_1, k_2 r_2 + b_2)$$

where $k\_1 > 0$, $k\_2 > 0$, $b\_1$, $b\_2$ are real numbers.

In my code, I simply define the similarity as the sum of Pearson
correlation coefficients of x and y, that is,

$$Sim(r_1, r_2) = Corr(r_1.x, r_2.x) + Corr(r_1.y, r_2.y)$$

The whole source code is (note that I use 500 recording points
instead of 50):

```r
library(grid);
getData = function()
{
    if(.Platform$OS.type == 'windows') x11() else x11(type = 'Xlib');
    pushViewport(viewport());
    grid.rect();
    px = NULL;
    py = NULL;
    mousedown = function(buttons, x, y)
    {
        if(length(buttons) > 1 || identical(buttons, 2L))
            return(invisible(1));
        eventEnv$onMouseMove = mousemove;
        NULL
    }
    mousemove = function(buttons, x, y)
    {
        px <<- c(px, x);
        py <<- c(py, y);
        grid.points(x, y);
        NULL
    }
    mouseup = function(buttons, x, y) {
        eventEnv$onMouseMove = NULL;
        NULL
    }
    setGraphicsEventHandlers(onMouseDown = mousedown,
                             onMouseUp = mouseup);
    eventEnv = getGraphicsEventEnv();
    cat("Click down left mouse button and drag to draw the number,
        right click to finish.\n");
    getGraphicsEvent();
    dev.off();
    s = seq(0, 1, length.out = length(px));
    spx = spline(s, px, n = 500)$y;
    spy = spline(s, py, n = 500)$y;
    return(cbind(spx, spy));
}
traceCorr = function(dat1, dat2)
{
    cor(dat1[, 1], dat2[, 1]) + cor(dat1[, 2], dat2[, 2]);
}

# Please set the proper path of this file.
load("train.RData");
guess = function(verbose = FALSE)
{
    test = getData();
    coefs = sapply(recogTrain, traceCorr, dat2 = test);
    num = which.max(coefs);
    if(num == 10) num = 0;
    if(verbose) print(coefs);
    cat("I guess what you have input is ", num, ".\n", sep = "");
}
guess();
```

To run the code, you must load the "training set", the file
`train.RData`, into R using the `load()` function, and then call
`guess()` to play with it.

Have fun!

[Download: Source code and training dataset](https://github.com/downloads/yixuan/en/Handwriting_recognition.zip)
