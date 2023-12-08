---
title: Hugo tips and tricks
date: 2023-11-27
draft: true
categories:
- publishing
tags:
- hugo
- jekyll
- github
showshowToc: true
---

This page describes my experiences with the static web site publishing tool [Hugo].

### Open link i new tab

See <https://discourse.gohugo.io/t/how-to-open-plain-url-links-in-a-new-tab/25523/3>

Create `layouts\_default\_markup\render-link.html` with:

~~~
<a href="{{ .Destination | safeURL }}"{{ with .Title}} title="{{ . }}"{{ end }}{{ if strings.HasPrefix .Destination "http" }} target="_blank" rel="noopener"{{ end }}>{{ .Text | safeHTML }}</a>
~~~



[Hugo]: {{<param "hugo_link">}}