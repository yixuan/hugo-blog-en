---
title: "Using system fonts in R graphs"
slug: "using-system-fonts-in-r-graphs"
date: 2014-01-01
categories:
- Programming
- R
tags:
- programming
- graphics
- font
- R
---

This is a pretty old topic in R graphics.
[A classical article in R NEWS](https://cran.r-project.org/doc/Rnews/Rnews_2006-2.pdf),
*Non-standard fonts in PostScript and PDF graphics*,
describes how to use and embed system fonts in the PDF/PostScript device.
More recently, [Winston Chang](https://github.com/wch) developed
the [extrafont](https://github.com/wch/extrafont) package, which
makes the procedure much easier. A useful introduction article can be found in the
[readme page](https://github.com/wch/extrafont/blob/master/README.md) of `extrafont`,
and also from the [Revolution blog](http://blog.revolutionanalytics.com/2012/09/how-to-use-your-favorite-fonts-in-r-charts.html).

Now, we have another choice: the `showtext` package.

[showtext](https://github.com/yixuan/showtext/) 0.2 has just been
[submitted to CRAN](https://cran.r-project.org/web/packages/showtext/index.html).
Below is the introduction of this package excerpted from the
[README.md](https://github.com/yixuan/showtext/blob/master/README.md) file. In short,

> We are now much freer to use system fonts in R to create figures.

## What's this package all about?
`showtext` is an R package to draw text in R graphs.

> Wait, R already has `text()` function to do that...

Yes, but drawing text is a very complicated task, and it always depends on
the specific **Graphics Device**.
(Graphics device is the engine to create images.
For example, R provides PDF device, called by function `pdf()`,
to create graphs in PDF format)
Sometimes the graphics device doesn't support text drawing nicely,
**especially in using fonts**.

From my own experience, I find it always troublesome to create PDF
graphs with Chinese characters. This is because most of the standard
fonts used by `pdf()` don't contain Chinese character glyphs, and
even worse users could hardly use the fonts that are already installed
in their operating system. (It seems still possible, though)

`showtext` tries to do the following two things:

- Let R know about these system fonts
- Use these fonts to draw text

## Why `pdf()` doesn't work and how `showtext` works
Let me explain a little bit about how `pdf()` works.

To my best knowledge (may be wrong, so please point it out if I make
mistakes), the default PDF device of R doesn't "draw" the text,
but actually "describes" the text in the PDF file.
That is to say, instead of drawing lines and curves of the actual glyph,
it only embeds information about the text, for example what characters
it has, which font it uses, etc.

However, the text with declared font may be displayed differently in
different OS. The two images below are the screenshots of the same PDF
file created by R but viewed under Windows and Linux respectively.

<div align="center">
  <img src="https://i.imgur.com/x1zM34F.png" />
</div>

This means that the appearance of graph created by `pdf()` is
system dependent. If you unfortunately don't have the declared font
in your system, you may not be able to see the text correctly at all.

In comparison, `showtext` package tries to solve this problem by
converting text into lines and curves, thus having the same appearance
under all platforms. More importantly, `showtext` can use system font
files, so you can show your text in any font you want.
This solves the Chinese character problem I mentioned in the beginning
because I can load my favorite Chinese font to R and use that to draw
text. Also, people who view this graph don't need to install the font
that creates the graph. It provides convenience to both graph makers
and graph viewers.

## The Usage
To create a graph using a specified font, you only need to do:

- (\*) Load the font
- Open the graphics device
- (\*) Claim that you want to use `showtext` to draw the text
- Plot
- Close the device

Only the steps marked with (\*) are newly added. Below is an example:

```r
library(showtext)
font.add("fang", "simfang.ttf") ## add font
pdf("showtext-ex1.pdf")
plot(1, type = "n")
showtext.begin()                ## turn on showtext
text(1, 1, intToUtf8(c(82, 35821, 35328)), cex = 10, family = "fang")
showtext.end()                  ## turn off showtext
dev.off()
```

<div align="center">
  <img src="https://i.imgur.com/u5uvjy5.png" />
</div>

The use of `intToUtf8()` is for convenience if you can't view or input
Chinese characters. You can instead use

```r
text(1, 1, "R语言", cex = 10, family = "fang")
```

This example should work fine on Windows. For other OS, you may not have
the `simfang.ttf` font file, but there is no difficulty in using something
else. You can see the next section to learn details about how to load
a font with `showtext`.

## Loading font
Loading font is actually done by package [sysfonts](https://github.com/yixuan/sysfonts/),
which is depended on by `showtext`.

The easiest way to load font into R is by calling `font.add(family, regular, ...)`,
where `family` is the name that you give to that font (so that later you can
call `par(family = ...)` to use this font in plotting), and `regular` is the
path to the font file. Usually the font file will be located in some "standard"
directories in the system (for example on Windows it is typically C:/Windows/Fonts).
You can use `font.paths()` to check the current search path or add a new one,
and use `font.files()` to list available font files in the search path.

Usually there are many free fonts that can be downloaded from the web and then used by
`showtext`, as the following example shows:

```r
library(showtext)

wd = setwd(tempdir())
download.file("http://fontpro.com/download-family.php?file=35701",
              "merienda-r.ttf", mode="wb")
download.file("http://fontpro.com/download-family.php?file=35700",
              "merienda-b.ttf", mode="wb")
font.add("merienda",
         regular = "merienda-r.ttf",
         bold = "merienda-b.ttf")
setwd(wd)

pdf("showtext-ex2.pdf", 7, 4)
plot(1, type = "n", xlab = "", ylab = "")
showtext.begin()
par(family = "merienda")
text(1, 1.2, "R can use this font!", cex = 2)
text(1, 0.8, "And in Bold font face!", font = 2, cex = 2)
showtext.end()
dev.off()
```

<div align="center">
  <img src="https://i.imgur.com/EUIGQ6L.png" />
</div>

In this case we add two font faces(regular and bold) with the family name
"merienda", and use `font = 2` to select the bold font face (`font = 1` is
selected by default, which is the regular font face).

At present `font.add()` supports TrueType fonts(\*.ttf/\*.ttc) and
OpenType fonts(\*.otf), but adding new
font type is trivial as long as FreeType supports it.

Note that `showtext` includes an open source CJK font
[WenQuanYi Micro Hei](http://wenq.org/wqy2/index.cgi?MicroHei%28en%29).
If you just want to show CJK text in your graph, you don't need to add any
extra font at all.

## Known issues
The image created by bitmap graphics devices (`png()`, `jpeg()`, ...)
looks ugly because they don't support anti-alias feature well. To produce
high-quality output, try to use the `CairoPNG()` and `CairoJPEG()` devices from the
[Cairo](https://cran.r-project.org/web/packages/Cairo/index.html) package.

## The internals of `showtext`
Every graphics device in R implements some functions to draw specific graphical
elements, e.g., `line()` to draw lines, `path()` and `polygon()` to draw polygons,
`text()` or `textUTF8()` to show text, etc. What `showtext` does is to override
their own text rendering functions and replace them by hooks provided in `showtext`
that will further call the device's `path()`, `polygon()` or `line()` to draw the
character glyphs.

This action is done only when you call `showtext.begin()` and won't modify the
graphics device if you call `showtext.end()` to restore the original device functions back.
