---
title: "A Brief But Comprehensive Book About TikZ and PGFPLOTS"
date: 2024-12-20T09:07:53+08:00
tags: ['latex', 'tikz', 'pgfplots', 'book']
draft: false
type: post
---

Earlier this year, I started a new GitHub repository called [TikZ_and_PGFPLOTS_draw](https://github.com/dbrg77/TikZ_and_PGFPLOTS_draw) where I used [TikZ](https://tikz.dev/) and [PGFPLOTS](https://pgfplots.sourceforge.net) to draw vector graphics for scientific illustrations. Now I'm gradually learning and switching to [Ipe](https://ipe.otfried.org/) and [Inkscape](https://inkscape.org) for a more flexible and easier drawing experience. However, TikZ and PGFPLOTS are the two things that I use routinely, so I still want to practice my skills. Therefore, I have kept updating the repo whenever I want to draw some basic stuff.

Even though I routinely use TikZ and PGFPLOTS, the reality is that I only remember a few key things that allow me to start with, such as drawing rectangles, circles, nodes, putting pictures into nodes, axis and some basic plots on the axis. I have never really learnt the those things systematically. When I started using TikZ/PGF and PGFPLOTS, I was just following the [Overleaf TikZ package webpage](https://www.overleaf.com/learn/latex/TikZ_package) and [A very minimal introduction to TikZ](https://cremeronline.com/LaTeX/minimaltikz.pdf), both of which were great. I managed to quickly start drawing after going through those examples and memorising a few commands. However, the logic behind the plot and some options of the commands were not entirely clear to me, and I often need to guess and use Google when I encounter a problem. [Mr. P Solver](https://www.youtube.com/watch?v=4NHqeNJbXVw) has a perfect video about how people like me use $\LaTeX{}$, TikZ and PGFPLOTS, and I quote from the video:

> You Google what you need.
> And then forget about it.
> Until the next time you need it.
> At which point you google it again.

The consequence is that I get better and better at what I already knew, and I even get some sort of physical memory in my fingers. For those things that I'm less familiar with, I still cannot remember how to do them.

Recently, I finished drawing a slightly more complicated illustrations of some key stages of mouse pre-implantation embryos, including the zygote, 2-cell, 4-cell, 8-cell, morula and blastocyst stages. The output and codes are in the [mouse_embryo_dev](https://github.com/dbrg77/TikZ_and_PGFPLOTS_draw/tree/main/mouse_embryo_dev) directory of the repo. During the making, I need to invoke some commands that are out of my regular usage, such as polar coordinates, loops and rotations. Of course, after some heavy Googling, I managed to get a fairly decent result.

During the internet search, I came across the [Unlocking LaTeX Graphics](https://www.youtube.com/@UnlockingLaTeXGraphics) YouTube channel by [Dr. Tamara G. Kolda](https://www.mathsci.ai). I was immediately hooked and ended up watching all the videos there non-stop. The videos were brief and extremely clear, showing just the right amount of details about different key aspects of TikZ with useful examples. It was enjoyable to watch as someone who is already familiar with the basics of $\LaTeX{}$. The channel is a companion to the book [*Unlocking LaTeX Graphics - A Concise Guide to TikZ/PGF and PGFPLOTS*](https://latex-graphics.com) by Dr. Kolda.

After watching the videos, I decided to buy the book. When I got the actual book, I was surprised about how thin the book is: only 109 pages (120 including appendices and index). It was very unexpected considering how long the manuals of [TikZ](http://mirrors.ctan.org/graphics/pgf/base/doc/pgfmanual.pdf) and [PGFPLOTS](http://mirrors.ctan.org/graphics/pgf/contrib/pgfplots/doc/pgfplots.pdf) are (over 1,800 pages in total). If you just read the book without actually trying the codes along the way, you can finish the book within just a few hours.

![](/images/2024-12-20/book_thickness.jpg)

The book is great for those who have some basic knowledge about $\LaTeX{}$ and want to draw quality figures that match the theme of the document. It starts with a concise introductions of **path** and **node** with many useful examples. Then it continues to introduce how to use PGFPLOTS to draw plots by reading external data files. There are also some advanced topics and nuances of TikZ and PGFPLOTS that are highly relevant for the daily usage as well. In the preface of the book, the author wrote:

> My goal with this book is to boil 1,800 pages down to something that can get you making graphics quickly, focusing on the 10% of TikZ and PGFPLOTS commands that users need 90% of the time.

On the back cover of the book, it says:

> This book is for novices just starting out as well as experts looking to better master these tools. The only prerequisite is a basic understanding of $\LaTeX{}$.

In my opinion, the author did a very good job on those things. With the book, I can get to know the basic principles behind TikZ/PGF and PGFPLOTS in a more systematic manner. For example, not until I read the book, did I realise that shapes like rectangles and circles were essentially **path**, and commands like `\draw`, `\fill` and `\filldraw` are shortcuts for `\path[draw]`, `\path[fill]` and `\path[fill,draw]`, respectively, which explains why certain things can be drawn in different ways and why some arguments are organised in certain ways. As a reductionist, I like this kind of knowledge.

{{< figure src="/images/2024-12-20/example1.jpg" width="75%" >}}

{{< figure src="/images/2024-12-20/example2.jpg" width="75%" >}}

One thing that I found nice and amusing is the **Oddity** call-outs in the book, where the author pointed out some problematic, inconsistent and counter-intuitive things associated with the commands for drawing. When I saw them, I can immediately relate to them. For example, the following two oddities caused me a lot of trouble and confusion when I first started:

![](/images/2024-12-20/callout1.jpg)

![](/images/2024-12-20/callout2.jpg)

Overall, the book is really great. It is comprehensive for almost all my daily usage and it provides just the right amount of explanations about everything. After reading the book, it made me want to clean up all my previous codes, lol.

