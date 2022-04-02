---
layout: post
title:  "WebSharper simple inputs"
categories: websharper
tags: input html5
---

Attempting to use math markdown in liquid.

For more information, see [Creating a mathematics bloc with Jekyll](https://medium.com/coffee-in-a-klein-bottle/creating-a-mathematics-blog-with-jekyll-78cdee0339f3).

Somehow, you have to insert these scripts into the default html.

~~~html
<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
<script id=”MathJax-script” async src=”https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>
~~~

Then it should be possible to use $$ x = y ^ 2 $$.

But I hate to touch the default layout, so need to find another way around.

Still unsolved, maybe can define a layout post-math which includes post.