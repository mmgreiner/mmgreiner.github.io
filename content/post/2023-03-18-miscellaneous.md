---
title: Miscellaneous
categories:
- programming
- publishing
date: "2023-03-18T00:00:00Z"
mermaid: true
tags:
- jekyll
- F#
- pandoc
- unix
showToc: true
---

This is a list of reminders and open questions I still wanted to look at in more detail.

## CSV conversion

Next to [FSharp.Data] there is [CSVHelper]. However, it does not support FSharp types intrinsically. A [stackoverflow](https://stackoverflow.com/questions/66401283/how-do-you-use-csvhelper-csvwriter-with-f-option-types) article describes the details.

I have concluded that to read CSV files, [FSharp.Data] is the best. To easily write CSV files, use [CSVHelper].

The configuration is set like this:

~~~fsharp
use csvStream = new IO.StreamWriter("woCancelled.csv")
let culture = System.Globalization.CultureInfo.InvariantCulture
let config = new CsvHelper.Configuration.CsvConfiguration(culture)
config.Delimiter <- ";"

use csvWriter = new CsvHelper.CsvWriter(csvStream, config)
csvWriter.WriteRecords(data)
~~~

[FSharp.Data]: https://fsprojects.github.io/FSharp.Data/
[CSVHelper]: https://joshclose.github.io/CsvHelper/

### Stress tests of CSV

There are several stress tests for CSV reading and writing:
- <https://github.com/maxogden/csv-spectrum>
- <https://github.com/wireservice/csvkit/tree/master/examples>
- <https://discourse.julialang.org/t/csv-reader-benchmarks-julia-reads-csvs-10-20x-faster-than-python-and-r/41974>
- <https://data.world/datasets/csv>

## Writing scientific papers

see <https://jaantollander.com/post/scientific-writing-with-markdown/>

See with more detail: 
<https://programminghistorian.org/en/lessons/sustainable-authorship-in-plain-text-using-pandoc-and-markdown>

## Print Markdown with pandoc

[pandoc] is a tool to convert markdown to all kinds of formats. It states:

> If you need to convert files from one markup format into another, pandoc is your swiss-army knife.

You can use the yaml header in a markdown file to tell pandoc how to print it. For instance:

~~~~yaml
---
title: My Title
author: Markus Greiner
date: 2024-01-18
fontsize: 12pt
papersize: a4
geometry: margin=2cm
---
~~~~

This will use paper size `A4` and font `12pt` when converting to pandoc, and will set the margin to 2 cm. If you want to print it landscape, do:

~~~yaml
---
geometry: landscape
---
~~~

The conversion to pdf is done as follows:

~~~~csh
% pandoc *.md -o mydoc.pdf
~~~~


[pandoc]: https://pandoc.org

## Humanizer

> [Humanizer] meets all your .NET needs for manipulating and displaying strings, enums, dates, times, timespans, numbers and quantities. It is part of the [.NET Foundation](https://www.dotnetfoundation.org/), and operates under their [code of conduct](https://www.dotnetfoundation.org/code-of-conduct). 

[Humanizer] is an amazing library to do string conversions from programming strings to human readable strings. For instance, it handles all the PascalCase, camelCase, etc renaming, shortening of strings, even creating `[Display("Family Name")]` attribute handling.


[Humanizer]: https://github.com/Humanizr/Humanizer

## Yaml

Use [YamlDotNet](https://github.com/aaubry/YamlDotNet).

## Markdown

Use [Markdig](https://github.com/xoofx/markdig).

## datatools

> [datatools](https://github.com/caltechlibrary/datatools/) is a rich collection of command line programs targetting data conversion, cleanup and analysis directly from your favorite POSIX shell. It has proven useful for data collaberations where individual members of a project may prefer different toolsets in their analysis (e.g. Julia, R, Python) but want to work from a common baseline. It also has been used intensively for internal reporting from various Caltech Library metadata sources.

### MVC, Razor, Razor Pages, MVC, and Blazor

I found a good article explaining all of this on [Progress Telerik](https://www.telerik.com/blogs/difference-between-blazor-vs-razor)

## Unix stuff

To only grep on stderr, do:

    go run . 2>&1 >/dev/null | grep MyPattern

