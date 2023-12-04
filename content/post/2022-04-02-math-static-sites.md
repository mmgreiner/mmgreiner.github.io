---
categories:
- publishing
- math
date: "2022-04-02T00:00:00Z"
tags:
- liquid
- math
- markdown
- mathjax
- jekyll
- hugo
- katex
title: Math formulas and static web site generators
math: true
showshowToc: true
---

## Mathematical formulas

While working on post [Mathe Uffzig]({{< ref 2022-04-02-mathe-uffzgi >}}), I stumbled upon the problem of displaying mathematical formulas in the [Jekyll] markdown templating engine [Liquid](liquid).

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

Now in the post that uses math scripts, include the line `{% include mathscripts.html %}`. That's it! 

## Math expressions in [Hugo]

Using the [Hugo] templating engine, we have to take a different approach. One that I found is using the [Katex] Javascript library for math expressions. 

I have found the following instructions to set up math expressions with Hugo, one from [this blog](https://mertbakir.gitlab.io/hugo/math-typesetting-in-hugo/) and another from the author of the [PaperMod theme](https://adityatelange.github.io/hugo-PaperMod/posts/math-typesetting/)

There are basically three steps involved:

1. create a partial that includes the necessary scripts
2. include those scripts in the header
3. Enable math either globally or per page.

### Create Partial

Create a partial under `/layouts/partials/math.html` with the following content (taken from [Katex Autorender extension](https://katex.org/docs/autorender.html)):

```
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/katex.min.css" integrity="sha384-n8MVd4RsNIU0tAv4ct0nTaAbDJwPJzDEaqSD1odI+WdtXRGWt2kTvGFasHpSy3SV" crossorigin="anonymous">
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/katex.min.js" integrity="sha384-XjKyOOlGwcjNTAIQHIpgOno0Hl1YQqzUOEleOLALmuqehneUG+vnGctmUb0ZY0l8" crossorigin="anonymous"></script>
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/contrib/auto-render.min.js" integrity="sha384-+VBxd3r6XgURycqtZ117nYw44OOcIax56Z4dCRWbxyPt0Koah1uHoK0o4+/RRE05" crossorigin="anonymous"></script>
<script>
    document.addEventListener("DOMContentLoaded", function() {
        renderMathInElement(document.body, {
          // customised options
          // • auto-render specific keys, e.g.:
          delimiters: [
              {left: '$$', right: '$$', display: true},
              {left: '$', right: '$', display: false},
              {left: '\\(', right: '\\)', display: false},
              {left: '\\[', right: '\\]', display: true}
          ],
          // • rendering keys, e.g.:
          throwOnError : false
        });
    });
</script>
```

Double dollar signs will be displayed as blocks, single dollar sign inline.


### Include scripts

Include the script in the `extend_head.html`, which should reside in `themes/.../layouts/partials/extend_head.html`:

````
{{ if or .Params.math .Site.Params.math }}
{{ partial "math.html" . }}
{{ end }}
````

### Include in front matter

In your page's front matter, include: 

````
---
...
math: true
---
````

feerr

### Math code blocks

One issue is, that all these pages are typically stored on [github](https://github.com/), which uses again it's own way how to display math equations. It seems that code blocks of type math are the future.

With version 0.93.0, [Hugo] allows extensions to the rendering of code blocks, see [render hooks](https://gohugo.io/templates/render-hooks/#render-hooks-for-code-blocks). 

You have to add the file `render-codeblock-math.html`.

    layouts
    ├── _default
    │   └── _markup
    │       └── render-codeblock-math.html
    ├── partials
    │   └── math.html

This file contains the following brief segment:

````
<p>
    {{- .Inner | safeHTML }}
</p>
````

Now the codeblock with language `math` will render to a paragraph with the math block expression inside (with double dollars), which will be picked up by [Katex].


### Linebreaks

I had some difficulties generating multi-line math equations. I ended up using `\cr` for a line break, since double backslash did not work.




[liquid]: https://shopify.github.io/liquid/
[mathjax]: https://www.mathjax.org
[kramdown]: https://kramdown.gettalong.org/quickref.html#code-blocks
[Jekyll]: {{< param "Jekyll_link" >}}
[Hugo]: {{< param "Hugo_link" >}}
[Katex]: https://katex.org
[github]: {{< param "github_link" >}}
