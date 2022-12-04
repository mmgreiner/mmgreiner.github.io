---
title: "Bulma hints"
category: programming
tags: html bulma
---

Give the simplest possible bulma html with two columns and a decent margin.

See [Bulma](https://bulma.io)

~~~html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Hello Bulma!</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bulma@0.9.4/css/bulma.min.css">
  </head>
  <body>
  <section class="section">
    <div class="container">
      <h1 class="title">
        Hello World
      </h1>
      <p class="subtitle">
        My first website with <strong>Bulma</strong>!
      </p>
    </div>
  </section>
  </body>
</html>
~~~

Questions:
- [] Should navigation go inside the container?
- [] For a Bolero application: how should ``index.fs`` look like?