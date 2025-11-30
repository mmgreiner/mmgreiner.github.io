---
title: Simple.css framework
date: 2025-01-01
draft: false
categories:
- programming
tags:
- "css framework"
- rails
- simplecss
showToc: false
---

After using [bulma] for some time, I moved on to an even easier framework, called [Simple.css]. All you have to do is to include it:

```html
<link rel="stylesheet" href="https://cdn.simplecss.org/simple.min.css">
```

A typical `application.html.erb` page would look like this:

```html
<!DOCTYPE html>
<html>
  <head>
    <title><%= content_for(:title) || "Title App" %></title>

    <%= stylesheet_link_tag "application" %>
    <%= stylesheet_link_tag "https://cdn.simplecss.org/simple.min.css" %>

  </head>

  <body>
    <header>
      <nav>
        <ul>
          <li><%= link_to :home, root_path %>
        </ul>
      </nav>
    </header>
    <main>
      <%= yield %>
    </main>
    <footer>
      Copyright © Greinersoft
    </footer>
  </body>
</html>
```

## Reduce margin

The standard template has a quite large margin. To reduce this margin, you can use:

```html
<style>
  :root {
    --reading-width: min(45rem, 90%);
  }
  body {
    grid-template-columns: 1fr 98% 1fr;
  }
</style>
```

Note that `fr` stands for fraction.

[Simple.css]: https://simplecss.org/
[bulma]: https://bulma.io/
