---
title: Ruby on Rails ticks and trips
description: Ruby on Rails with bulma, haml, etc.
date: 2024-11-12
categories:
- programming
tags:
- ruby
- rails
- haml
- bulma
---

I have been using [Ruby on Rails][rails] quite a lot lately due to its extremely rapid way to build proof of concepts for web applications.

However, I'm typically using [haml] for the view templates, and [bulma] as css framework.

## Haml

[haml](https://haml.info/) is a templating engine with the goal:

> Haml (HTML abstraction markup language) is based on one primary principle: markup should be beautiful. 

It replaces HTML templates like:

~~~~html
<section class="container">
  <h1><%= post.title %></h1>
  <h2><%= post.subtitle %></h2>
  <div class="content">
    <%= post.content %>
  </div>
</section>
~~~~

with 

~~~~haml
%section.container
  %h1= post.title
  %h2= post.subtitle
  .content
    = post.content
~~~~

The [haml] gem is quite well documented. To use it in rails, first, install it and the helper tool:

    % bundle add haml-rails
    % bundle add html2haml

Then, you can convert the exising `.html.erb` files to `.html.haml` files:

    $ rails generate haml:application_layout convert

Finally, you have to remove the `application.html.erb` file. Or, you can convert all your *erb* files to *haml*:

    $ rails haml:erb2haml

This will also ask you if you want to delete all the *erb* files.


## Bulma

I picked [bulma] as the css framework after many years of working with [purecss](https://pure-css.github.io/) mainly becouse [bulma] offered better responsive designs, particularly when it came to responsive navigation menus.

To use [bulma] with [rails], you have to do the following modifications:

Add the stylesheet to the  header:

~~~~haml
%html
  %head
    %meta{:content => "text/html; charset=UTF-8", "http-equiv" => "Content-Type"}/
    ...
    = stylesheet_link_tag "https://cdn.jsdelivr.net/npm/bulma@1.0.2/css/bulma.min.css"
~~~~

Then use `container` and apply it to all the elements of the body:

~~~~haml
%body
%section.section        # added
  .container            # added
    = yield
~~~~

That's it. Now you have the default fonts and colors of [bulma].

Obviously, if you want to make forms look nicer you have to change the auto-generated erb / haml files. A good guide is at the [bulma forms](https://bulma.io/documentation/form/general/) documentation.


[rails]: https://guides.rubyonrails.org/index.html
[haml]: https://github.com/haml/haml-rails
[bulma]: https://bulma.io/