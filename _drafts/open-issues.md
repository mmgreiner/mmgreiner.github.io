---
title:  "Open questions"
categories: programming jekyll
tags: jekyll
toc: true
---

This is a list of reminders and open questions I still wanted to look at

## Mermaid

> [Mermaid] is a JavaScript based diagramming and charting tool that renders Markdown-inspired text definitions to create and modify diagrams dynamically.

It can handle

- FLowcharts
- Sequence Diagram
- Class Diagram
- ER Diagram
- User Journey
- Gantt
- Pie Chart
- Requirement Diagram
- Gitgraph Diagram
- C4C Diagram

Exmple Sequence Diagram:


    sequenceDiagram
        participant Alice
        participant Bob
        Alice->>John: Hello John, how are you?
        loop Healthcheck
            John->>John: Fight against hypochondria
        end
        Note right of John: Rational thoughts <br/>prevail!
        John-->>Alice: Great!
        John->>Bob: How about you?
        Bob-->>John: Jolly good!



It comes with [Github plugins](https://mermaid-js.github.io/mermaid/#/./integrations?id=productivity) and [Jekyll plugin](https://mermaid-js.github.io/mermaid/#/./integrations?id=other)


[Mermaid]: https://mermaid-js.github.io/mermaid/#/

[![](https://mermaid.ink/img/pako:eNptUcFOwzAM_RWTcyfuFSoCIZgmwQGuvbiJ10RL45I6oGrav5NmsEoTPjl-79l5ekel2ZCq1USfiYKmJ4d9xKEN8FsjRnHajRgEHrzT9D_0yN0KFN6maXZsQw1b8p5h6Suw_A0YCWZO9yvfM4-Zhl6stqQPK7LUorwse3a9FcAeXZgE7DyythxMdLiKKJj18cZCEIuK93Be8o7iOKAHsZwyMsFdF2-bMdIXOn-zisvpfLsYquElEso13DTZe3a5OOs4yZW3DG4uv9-x9zP0zKZsUZUaKA7oTE7guExaJZYGalWdW4Px0Ko2nDIPk_DHHLSqJSaqVBoNyl9aqt6jn_KUjBOOr-dIS7KnH2Tqlh8?type=png)](https://mermaid.live/edit#pako:eNptUcFOwzAM_RWTcyfuFSoCIZgmwQGuvbiJ10RL45I6oGrav5NmsEoTPjl-79l5ekel2ZCq1USfiYKmJ4d9xKEN8FsjRnHajRgEHrzT9D_0yN0KFN6maXZsQw1b8p5h6Suw_A0YCWZO9yvfM4-Zhl6stqQPK7LUorwse3a9FcAeXZgE7DyythxMdLiKKJj18cZCEIuK93Be8o7iOKAHsZwyMsFdF2-bMdIXOn-zisvpfLsYquElEso13DTZe3a5OOs4yZW3DG4uv9-x9zP0zKZsUZUaKA7oTE7guExaJZYGalWdW4Px0Ko2nDIPk_DHHLSqJSaqVBoNyl9aqt6jn_KUjBOOr-dIS7KnH2Tqlh8)


## CSV conversion

Next to [FSharp.Data](https://fsprojects.github.io/FSharp.Data/) there is [CSVHelper](https://joshclose.github.io/CsvHelper/). However, it does not support FSharp types intrinsically. There is a stackoverflow article here: [stackoverflow](https://stackoverflow.com/questions/66401283/how-do-you-use-csvhelper-csvwriter-with-f-option-types).

The configuration is set like this:

~~~fsharp
use csvStream = new IO.StreamWriter("woCancelled.csv")
let culture = System.Globalization.CultureInfo.InvariantCulture
let config = new CsvHelper.Configuration.CsvConfiguration(culture)
config.Delimiter <- ";"

use csvWriter = new CsvHelper.CsvWriter(csvStream, config)
csvWriter.WriteRecords(data)
~~~

### Stess tests of CSV

See https://github.com/maxogden/csv-spectrum

See https://github.com/wireservice/csvkit/tree/master/examples

See https://discourse.julialang.org/t/csv-reader-benchmarks-julia-reads-csvs-10-20x-faster-than-python-and-r/41974

See https://data.world/datasets/csv

## Writing scientific papers

see <https://jaantollander.com/post/scientific-writing-with-markdown/>

See with more detail: 
<https://programminghistorian.org/en/lessons/sustainable-authorship-in-plain-text-using-pandoc-and-markdown>



