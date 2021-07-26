---
title: "A conversation with Hadley Wickham"
date: 2013-09-27
categories:
- Statistics
- R
tags:
- statistics
- data science
- ggplot2
- database
- SQL
- github
- RStudio
- data tidying
- programming
---

<img src="https://i.imgur.com/EPgIMLi.jpg" class="align-right"/>

> Dr. Hadley Wickham is the Chief Scientist of RStudio and Assistant Professor
of Statistics at Rice University. He is the developer of the famous R package `ggplot2`
for data visualization and the author of many other widely used packages like `plyr`
and `reshape2`. On Sep 13, 2013 he gave a talk at Department of Statistics,
Purdue University, and later I (Yixuan) had a conversation with him (Hadley), talking
about his own experience and interest on data visualization, data tidying, R programming
and other related topics.

> Below is the written record of our conversation, with a Chinese translation posted in
[Capital of Statistics](https://cosx.org/), the largest online community on Statistics in China.


**Yixuan:** Can you first tell us, how did you choose to enter the field of
Statistics and data science?

**Hadley:** I got my first degree from a medical school, and actually I was in the
half way to be a doctor (laugh). But I realized I didn't want to be a
doctor, so I went back to what I enjoyed in high school which was computer
science and statistics. I really liked programming and statistics,
and then did PhD in the United States, choosing data visualization
and multivariate data analysis as my thesis topic.

**Yixuan:** How do you define data scientist compared to statistician?

**Hadley:** I think they are basically the same. Data scientists may mind more
about databases, more about programming, but basically they both
tried to do the same thing.

**Yixuan:** So it's like that data scientists are more involved in data in practice?

**Hadley:** Yeah. Traditionally statistics focuses much on the mathematical side.
A mathematical background is as important as a programming background.
To some extent it doesn't matter if you know the right thing to do but
you can't actually tell the computer how to do it. But equally
it doesn't matter if you can tell a computer what to do but you don't
know what you are doing. (laugh)

**Yixuan:** Can you illustrate the most exciting thing and most challenging thing
in your work?

**Hadley:** The thing I'm most excited about now is the next version of
[`ggplot2`](http://ggplot2.org/),
which is called [`ggvis`](https://github.com/rstudio/ggvis).
It goes around with graphics and interactivity.
I'm working on that with Winston Chang, who is also in RStudio.
Hopefully we can have something to show by the end of the year.

**Yixuan:** So that's a long term plan for the `ggplot2`?

**Hadley:** Yeah. Basically it is pretty obvious that for data visualization now
you want to be doing them on the web, because everyone has a web
browser. And another thing is that the people who spend the most time
making graphics in general very fast across every platform are the
browser makers. There is a lot of competition between Chrome
and Firefox, about who can be the faster. And a lot of that now is
making it possible to do interactive graphics and statistical graphics
really quickly, much more than you can do in the past. That is a really
important principle to make graphics and also to make it easy to add
interactivity to graphics. For example, you can add a slider that
automatically changes the bin of a histogram, or the span of a loess
smoother. So it's pretty fun working on that.

**Yixuan:** What about the most challenging thing?

**Hadley:** Another thing I'm working on the moment is
[`dplyr`](https://github.com/hadley/dplyr), which is the next
iteration of [`plyr`](http://plyr.had.co.nz/).
I have to learn a lot about how to write efficient
SQL. If you asked me two weeks ago about how much SQL I knew, maybe I
would say 75% of it. But now, after I've used it, I realize that I only
understand about 25% of it. It is much much richer and more complicated
than I realized. That's both challenging and fun to learn it.

**Yixuan:** OK. So from my own point of view, previously you are most famous for
the `ggplot2` package. And now we see you are paying more effort on some
data tidying tools like `plyr` and `reshape2`, and you also wrote some
tutorials about high performance computing using `Rcpp`. So how are these
techniques related to each other? The data visualization, the computing,
and data tidying?

**Hadley:** What I'm interested in is how to make data analysis easy and fast. So
just look at how much time you spend doing each part of data analysis.
If you spend 8 hours doing data cleaning and data tidying, but 2 hours
doing modeling, then you want to make the process faster. Obviously
you try to figure out how to make data tidying and data cleaning
faster at this time. Just like my talk today, we may find that the two
bottlenecks are what you want to do, and how you tell the computer to
do that. A lot of my existing work, like `ggplot2`, `plyr` and `reshape2` have
been more about how you make it easier to express what you want, not
how you make the computer fast.

Now it's easier to do all these sort of things, and the bottleneck is
actually doing the computation. Now I'm trying to learn how to write
fast code, how to write efficient R code, and how to connect to C++ to
achieve more speed. It's a kind of process to keep going around. If
the bottleneck is here, then I go to fix this one. Now it takes less
time, and the bottleneck shifts over there, then I work on that
problem.

**Yixuan:** So you are trying to reduce both the time in describing data, and also
the time in computing part.

**Hadley:** Right. And another thing I'm interested in is generally ...
I know I cannot write every R package that people need, so how can I
make it easier for other people to write good R code, and to write R
packages?

**Yixuan:** That's the [`devtools`](https://github.com/hadley/devtools)?

**Hadley:** Yeah, just make it easier to make other people to use R and contribute.

**Yixuan:** Can you introduce your toolbox in data analysis, about the softwares
and languages you use?

**Hadley:** I'm now pretty much an RStudio user. I used to use Sublime Text in the
Mac, but I've anyway shifted in the last couple of months. Fow now
RStudio is just easier to get around functions. I've spent 90%
of my time inside R. My job is analyzing data, and I'm also trying to
figure out like "what do I think people are trying to do", "what are
people struggling with", "how can I make it easier to express in R",
and "how can I make the code more efficient". So I still write mostly
in R, but if I discover bottleneck, then I may write C++ to make it
faster. The challenge is that you can write the code much much faster,
but it takes much much longer time to write. If I make a mistake, it's
likely to crash R, and you need to get started from the scratch,
a little bit annoying. But at least now, if R crashes, RStudio will
just restart R and keep going.

**Yixuan:** Many of the visitors of [our website](https://cosx.org/)
are curious about the dynamic
graphics. They want to know whether you have any plan to integrate
for example R and the D3 library in the next generation of `ggplot2`.
Any plan or progress about that?

**Hadley:** `ggvis` works by generating [Vega](http://trifacta.github.io/vega/) code,
and Vega is a library built on
top of D3. So `ggvis` is very much like built on top of that, and also
supports dynamic and interactive graphics. I may show you a demo of
ggvis.

(Showing the demo)

**Yixuan:** Another major change of software development we can see these years
is that social coding becomes more popular. Many developers have a
[Github](https://github.com/‎) account, for example. Do you think that will change the way we
develop R and related packages?

**Hadley:** I think so. Certainly I find that the time between creating a Github
repository and my first pull request is getting smaller and smaller.
Recently I created a new repository, and I didn't tell anyone about
that. After four hours there was a pull request. I think one of the
really nice thing about social coding is that authors get motivated
because you can see other people not only using it but also caring
about it, which is really really cool. We've talked a little bit
about how we can make use of [Gists](https://gist.github.com/).
One example is [RPubs](https://rpubs.com/). That should be based
on Gist, so you can fork someone else's work and add some
modification, and if they want they can pull the changes back -- we have a lot
of ideas around that.

Another thing I was doing lately was trying to figure out the best way
of reading a file, and R just gave one answer. I wrote an R Markdown
document providing three methods, and then I did a little bit of
benchmarking to see which one is faster. I tweeted about it, and people
forked and suggested other ways which were even faster. That's
a really good way to learn, like "here is my best effort, can you do
any better?" Any time when there are two people collaborating to write
a piece of code, it is almost always better than just one person.

**Yixuan:** And as the chief scientist in [RStudio](https://www.rstudio.com/),
do you have any future plan for RStudio?

**Hadley:** I'm looking forward to the day when there are more scientists (laugh).
I think when we start making money, we will start investing on the top
of the R community. One thing that we would like to do is how to make
R as a language faster and more efficient.

I'm also really interested in statistical learning as a family of
modeling techniques, a kind of fitting them together very well and
forming a grammer of models. You can make a new model by joing things
together, just like the grammer of graphics by which you can come up
with a new graphic that is just a new arrangment of existing components.
I think that's something that makes it easier to learn modeling.
For example, you can learn a linear model in this way, a random forest
in that way, and you can learn them in a unified framework.

Another thing I'd like to think is the Lasso-type method. In one of my
classes, I want to show that now you should always try stablized
regression, you should always try to do Lasso and the similar.
I think there are 13 packages that would do Lasso, and I tried them
all. But every single one of them broke for a different reason. For
example, it didn't support missing values, it didn't support
categorical variables, it didn't do predictions and standard errors,
or it didn't automatically find the lambda parameter. Maybe that's
because the authors are more interested in the theoretical papers,
not in providing a tool that you can use in data analysis. So I want
to integrate them together to form a tool that is fast and works well.

**Yixuan:** For our team ([Capital of Statistics]((https://cosx.org/))),
we have translated your ggplot2 book,
and Winston's R Graphics Cookbook is also ongoing. Can you introduce your next book,
if I'm correct, the [Advanced R Programming](http://adv-r.had.co.nz/)?

**Hadley:** The goal of the Advanced R Programming is basically to help people
become better R programmers. Lots and lots of books are about how to
do statistics with R, but not many about programming in R. Matloff's
[Art of R Programming](http://nostarch.com/artofr.htm) is a kind of
good basic and intermediate book,
and what I want to introduce are some of the features of the R
language that I think are really cool and powerful. To learn how to
use it you need to read a lot of documentation and I do a lot of
experiments to tell how things work. So I'm really interested in
helping people understand and write more efficient and also more
expressive code.

I think R has a reputation for being a horrible programming language,
but that's not really true. I think the heart of R is really a
beautiful and elegant language. The majority of people using R are not
programmers, so there is really elegant core as well as very tedious
R code. I think R is like javascript. There is a book called
[JavaScript: The Good Parts](http://shop.oreilly.com/product/9780596517748.do),
that tries to pull out that part.
The goal of my book is similar, not just telling people how to write
R in the elegant way, but also to make it easier for them to solve
problems, by introducing a little bit more the theory that underlies R.

**Yixuan:** The last question: what are your hobbies when you are not working?

**Hadley:** I like to cook. Recently I've been doing grilling, learning American
barbecue food. I also like to make cocktails.

**Yixuan:** OK. Thank you for the conversation!

<p><img src="https://i.imgur.com/ICvLmEQ.jpg" class="aligncenter"/></p>

(Hadley Wickham with his *ggplot2* book in Chinese translation)
