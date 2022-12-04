---
title:  "Open questions"
categories: programming
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