---
title: "Blog Not Down"
slug: "blog-not-down"
date: 2017-08-13
categories:
- Programming
tags:
- Hugo
- blogdown
- markdown
- dynamic document
- theme
---

It has been one year since my last article, and here is a quick post indicating
that my blog is not down. Instead, it has a new look thanks to
[blogdown](https://github.com/rstudio/blogdown). Yes, pun intended. :-)

**blogdown**, mostly written by [Yihui](https://yihui.name/), is an R
package that can help you rapidly create a static blog or website.
The package name has nothing to do with the status of a website
(as in "the server is down"), but rather follows the convention of other
[Markdown](https://daringfireball.net/projects/markdown/syntax)-based packages
such as [rmarkdown](https://github.com/rstudio/rmarkdown) and
[bookdown](https://github.com/rstudio/bookdown). (As for the name of *Markdown*,
I suspect that it was chosen such that it looked differently from other
popular _markup_ languages at that time such as HTML.)

The blogdown package is based on the [Hugo](https://gohugo.io/) blogging system.
Compared with the [Jekyll](https://jekyllrb.com/) system that my old blog was based on,
Hugo has some nice features that finally drive me to complete this switch:

1. The installation is very easy. In fact, it provides a single executable file for most
mainstream operating systems.
2. The blogdown package simplifies this process even more.
3. Hugo has a decent [theming system](https://themes.gohugo.io/), and more importantly,
it provides a mechanism with which you can import themes developed by others and make
your own modifications without messing up the code.

As I mentioned above, blogdown simplifies the procedure of installing Hugo, downloading
a theme, and creating a new site. The following three lines are sufficient for initiating
a new blog:

```r
library(blogdown)
install_hugo()  ## run once
new_site(theme = "kakawait/hugo-tranquilpeak-theme")
```

The `theme` parameter is optional since it has a default value `theme = "yihui/hugo-lithium-theme"`. Then you get a directory of Hugo source files, and
the generated static HTML pages in the `public` folder.

Note that unlike Jekyll blogs that can be directly rendered by [Github Pages](https://pages.github.com/), Hugo blogs are not yet supported by Github. But
fortunately, you can deploy your blog on [Netlify](https://www.netlify.com/) by
linking a Github repository.
There is a nice [article](https://www.netlify.com/blog/2016/09/21/a-step-by-step-guide-victor-hugo-on-netlify/) talking about this.

For me, the next major steps to fully migrate the blog to Hugo were as follows:

1. Copying `*.md` files to the `content` folder of Hugo site.
2. Making my customizations of the theme under the `layouts` folder. The layout
files under this folder have higher priority than the imported theme files, so you can
easily override some theme functions without polluting the upstream.
3. Files in the `static` folder will be located in the website root directory, for example CSS and images.
4. Push the directory to Github and link to Netlify.
5. Point the domain name to the new server address as described in [this document](https://www.netlify.com/docs/custom-domains/).

So far all of my posts are plain Markdown files, meaning that they are not dynamic
documents that contain executable code. But in fact another attractive feature of
blogdown is to allow you writing R-Markdown-style blogs that can be automatically
compiled into Markdown with the rendered output. At the time of writing the blogdown
package is under active development, and hope its first formal release goes to CRAN soon.
