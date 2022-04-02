---
layout: post
title:  "Liquid templating tips & tricks"
categories: jekyll
tags: liquid math markdown mathjax
---

{% include mathscripts.html %}

## Mathematical formulas

While working on post [Mathe Uffzig]({% post_url 2022-04-02-mathe-uffzgi %}), I stumbled upon the problem of displaying mathematical formulas in the Jekyll markdown templating engine [Liquid](liquid).

Searching the web, I found this blog: [Creating a mathematics blog with Jekyll](https://medium.com/coffee-in-a-klein-bottle/creating-a-mathematics-blog-with-jekyll-78cdee0339f3).

The basic ideas is to use [MathJax][mathjax] to visualize the formulars. This JavaScript has to be included in the layouts. 

To do so, I found two ways: Changing the `default.html` page or including the scripts locally.

### Changing default.html

This is explained in more detail in the [Jekyll documentation on layouts](https://jekyllrb.com/docs/layouts/).

**First**, get the `default.html` file from the theme folder. to open this, use:

~~~sh
# on unix
open $(bundle info --path minima)
~~~

Then create a folder `_layouts` and the file `default.html`. Copy from the minima the default.html file to into here and **add two lines**:

~~~html
{% raw %}<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
<script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>{% endraw %}
~~~

Now, **restart** the jekyll server. 

You can now insert `$$ x = y ^ 2 $$` into your markdown and it displays as a nice math formula:

$$ x = y^2 $$

### Local include

Another way, which doesn't require any changes to `default.html`, is with include.

Create the file `_includes/mathscripts.html` with the following content:

~~~html
{% raw %}<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
<script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>{% endraw %}
~~~

Now in the post that uses math scripts, include the line `{% raw %}{% include mathscripts.html %}{% endraw %}`. That's it! 

## Copy code to clipboard

See the post [Copy to Clipboard](https://www.aleksandrhovhannisyan.com/blog/how-to-add-a-copy-to-clipboard-button-to-your-jekyll-blog/) by Aleksandr Hovhannisyan.


[liquid]: https://shopify.github.io/liquid/
[mathjax]: https://www.mathjax.org