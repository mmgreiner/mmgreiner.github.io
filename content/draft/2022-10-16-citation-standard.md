---
categories:
- authoring
date: "2022-10-16T00:00:00Z"
draft: true
tags:
- bibTeX
title: BibTeX for standards
---

How to handle a standard?

[citation-guide] recommends `manual`.

    @manual{standard_1968,
    title         = "Letter Symbols for Quantities",
    howpublished  = "ANSI Standard Y10.5",
    year          = "1968"
    }


This [Blogspot](https://b-p-i.blogspot.com/2012/08/cite-iso-standard-bibtex.html) recommends `techreport`:

    @techreport{ISOPDF,
    author = {ISO},
    Institution = {International Organization for Standardization},
    address = {Geneva, Switzerland},
    Title = {Document management---{P}ortable document format---{P}art~1: {PDF}~1.7},
    number = {32000\char"2012 1:2008},
    Type = {ISO},
    Year = {2008}
    }

or 

~~~
{% raw %}@techreport{ISO13586,
type = {Standard},
key = {ISO 13586:2000(E)},
month = mar,
year = {2000},
title = {{Plastics -- Determination of fracture toughness (${G}_{\mathrm{IC}}$ and ${K}_{\mathrm{IC}}$) -- Linear elastic fracture mechanics ({LEFM}) approach}},
volume = {2000},
address = {Geneva, CH},
institution = {International Organization for Standardization}
}{% endraw %}
~~~

See also this [Blog]({{< ref 2022-10-13-bibliography >}})

[citation-guide]: https://libguides.nps.edu/citation/ieee-bibtex



## BibTeX generator

- https://truben.no/latex/bibtex/

