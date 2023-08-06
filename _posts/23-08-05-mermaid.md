---
title:  "Mermaid"
categories: programming jekyll
tags: jekyll mermaid
toc: false
---


> [Mermaid] is a JavaScript based diagramming and charting tool that renders Markdown-inspired text definitions to create and modify diagrams dynamically.

It can handle

- Flowcharts
- Sequence Diagram
- Class Diagram
- ER Diagram
- User Journey
- Gantt
- Pie Chart
- Requirement Diagram
- Gitgraph Diagram
- C4C Diagram

Example Sequence Diagram:

~~~
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
~~~


## Github support

Github now natively supports [mermaid], see this [article](https://github.blog/2022-02-14-include-diagrams-markdown-files-mermaid/). All you have to do is to mark a code block with **mermaid**:

~~~mermaid
graph TD 
A[Client] --> B[Load Balancer] 
B --> C[Server1] 
B --> D[Server2]
~~~

test 2: try to find a solution that works locally and on Github and Github pages.

<pre class="mermaid mermaid-language">
graph TD 
A[Client] --> B[Load Balancer] 
B --> C[Server1] 
B --> D[Server2]
</pre>

However, this will not work in your local Jekyll deployment. Also, it does not work on **Github Pages**. There you need the same approach as described [below](#using-mermaid-locally).

## Using mermaid locally

I looked at the [Github plugins](https://mermaid-js.github.io/mermaid/#/./integrations?id=productivity) and [Jekyll plugin]. However, they don't work in github. So I followed the [Mermaid HTML Tutorial](https://mermaid.js.org/intro/n00b-gettingStarted.html).

- Define a the file `mermaid.html` in `_includes`:

~~~html
<script type="module">
      import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.esm.min.mjs';
      mermaid.initialize({ startOnLoad: true });
</script>
~~~

{% include mermaid.html %}

- in the relevant posts, include it: {% raw %}{% include mermaid.html %}{% endraw %}.

- Now mark the diagramm with `<pre class="mermaid"`. Unfortunately, normal code blocks translate to `class="language-mermaid"`, which is not caught by the [mermaid] initialize function.

Example: 

~~~html
<pre class="mermaid">
            graph TD 
            A[Client] --> B[Load Balancer] 
            B --> C[Server1] 
            B --> D[Server2]
</pre>
~~~

Leads to this graph:

<pre class="mermaid">
            graph TD 
            A[Client] --> B[Load Balancer] 
            B --> C[Server1] 
            B --> D[Server2]
</pre>

### Online Editor

There is an online editor which allows you to draw diagrams and then store them and reference them as a picture - the markdown link can be downloaded. The following graph has been generated like this:

[![](https://mermaid.ink/img/pako:eNptUcFOwzAM_RWTcyfuFSoCIZgmwQGuvbiJ10RL45I6oGrav5NmsEoTPjl-79l5ekel2ZCq1USfiYKmJ4d9xKEN8FsjRnHajRgEHrzT9D_0yN0KFN6maXZsQw1b8p5h6Suw_A0YCWZO9yvfM4-Zhl6stqQPK7LUorwse3a9FcAeXZgE7DyythxMdLiKKJj18cZCEIuK93Be8o7iOKAHsZwyMsFdF2-bMdIXOn-zisvpfLsYquElEso13DTZe3a5OOs4yZW3DG4uv9-x9zP0zKZsUZUaKA7oTE7guExaJZYGalWdW4Px0Ko2nDIPk_DHHLSqJSaqVBoNyl9aqt6jn_KUjBOOr-dIS7KnH2Tqlh8?type=png)](https://mermaid.live/edit#pako:eNptUcFOwzAM_RWTcyfuFSoCIZgmwQGuvbiJ10RL45I6oGrav5NmsEoTPjl-79l5ekel2ZCq1USfiYKmJ4d9xKEN8FsjRnHajRgEHrzT9D_0yN0KFN6maXZsQw1b8p5h6Suw_A0YCWZO9yvfM4-Zhl6stqQPK7LUorwse3a9FcAeXZgE7DyythxMdLiKKJj18cZCEIuK93Be8o7iOKAHsZwyMsFdF2-bMdIXOn-zisvpfLsYquElEso13DTZe3a5OOs4yZW3DG4uv9-x9zP0zKZsUZUaKA7oTE7guExaJZYGalWdW4Px0Ko2nDIPk_DHHLSqJSaqVBoNyl9aqt6jn_KUjBOOr-dIS7KnH2Tqlh8)

### Minimum index.html

A minimum [Mermaid] project looks like this in `index.html`:

~~~~html
<html>
  <body>
    Here is one mermaid diagram:
    <pre class="mermaid">
            graph TD 
            A[Client] --> B[Load Balancer] 
            B --> C[Server1] 
            B --> D[Server2]
    </pre>

    And here is another:
    <pre class="mermaid">
            graph TD 
            A[Client] -->|tcp_123| B
            B(Load Balancer) 
            B -->|tcp_456| C[Server1] 
            B -->|tcp_456| D[Server2]
    </pre>

    <script type="module">
      import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.esm.min.mjs';
      mermaid.initialize({ startOnLoad: true });
    </script>
  </body>
</html>
~~~~

## Plugins

There are two plugins: 

- [jekyll-mermaid](https://rubygems.org/gems/jekyll-mermaid) of 2014
- [jekyll-mermaid-diagrams](https://github.com/fuzhibo/jekyll-mermaid-diagrams) of 2019.

Unfortunately, both don't work on github.

[Jekyll plugin]: https://mermaid-js.github.io/mermaid/#/./integrations?id=other
[Mermaid]: https://mermaid-js.github.io/mermaid/#/

## Conclusio
If publishing on github anyway, go with the [language code approach](#github-support) and ignore, that they are not displayed locally.


