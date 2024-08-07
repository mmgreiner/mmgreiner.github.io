---
title: Hugo tips and tricks
date: 2023-11-27
draft: false
categories:
- publishing
tags:
- hugo
- jekyll
showToc: true
---

This page describes my experiences with the static web site publishing tool [Hugo].

## Open link in new tab

There is a discussion which describes [How to open links in a new tab](https://discourse.gohugo.io/t/how-to-open-plain-url-links-in-a-new-tab/25523/3).

You need to create `layouts\_default\_markup\render-link.html` with:

~~~
<a href="{{ .Destination | safeURL }}"{{ with .Title}} title="{{ . }}"{{ end }}{{ if strings.HasPrefix .Destination "http" }} target="_blank" rel="noopener"{{ end }}>{{ .Text | safeHTML }}</a>
~~~

Then, in your markdown, when the link is prefixed with `http`, they will be opened in a new tab.

## HTML on markdown page

Normally, HTML on a markdown page is not recognized and you are advised to create a [shortcode](https://gohugo.io/content-management/shortcodes/) for it.

However, sometimes you need a table with more complex, multi-line cell entries. Then HTML tables come handy. 

First, you have to configure the Goldmark markdown processor to allow it. Change `hugo.yaml` as follows:

~~~~yaml
markup:
  goldmark:
    renderer: 
      unsafe: true
~~~~

Then create a table as follows; attention that there is an empty line after the `<td>` element:

~~~~
<table>
<thead>
<tr>
<th>Code</th>
</tr>
</thead>
<tbody>
<tr>
<td>

~~~ruby
def x(a)
end
~~~

</td>
</tr>
</tbody>
</table>
~~~~

For an example, see the [Ruby Rails Hints]() page.


## Generate JSON output

Normally, HUGO generates HTML output. However, it is also possible to generate other output formats like JSON.

See [Hugo custom output formats](https://gohugo.io/templates/output-formats/)

~~~~yaml
outputs:
    home:
        - HTML
        - RSS
        - JSON # is necessary
~~~~


## Including source code

What if you want to include source code in a code example? The post of [Marcus Olsson](https://marcusolsson.dev/how-to-include-code-examples-from-file-with-hugo/) shows how to do it.

Basically, you need to define a shortcode like this one I use, called `layout/shortcodes/include-html-code.html`:

~~~
{{< include-file file="layouts/shortcodes/include-file.html" >}}
~~~

The minuses after the curly brackets indicate that the spaces and line breaks are not kept.

Then in your post, use it as follows:

~~~html
{{</* include-file file="static/assets/purecss_index.html" */>}}
~~~

**Note**: Escaping shortcodes was kaken from [Chris Liatas](https://liatas.com/posts/escaping-hugo-shortcodes/)


[Hugo]: {{<param "hugo_link">}}