---
categories:
- programming
date: "2024-07-27"
draft: false
tags:
- html
- purecss
- fontawesome
title: A vertical responsive menu with PureCSS
showToc: false
---

I have been using [Pure.css] in most of my applications since it is really simple to use.

This is a small code sample that uses [Pure.css] with a responsive, vertical menu with hamburger and a submenu.

So it posed two challenges:
- responsive vertical menu with hamburger
- submenus

The given code is available on my [github].

## Submenus

[Pure.css] does a fairly good job explaining how to do submenus in their documentation [Vertical Menu with Submenu](https://purecss.io/menus/#vertical-menu-with-submenus)

Unfortunately, this code did not show the submenu. It took me some time to understand the line:

~~~html
<div class="pure-menu custom-restricted-width">
  <ul class="pure-menu-list">
  ...
</div>
~~~

The trick lies in the `custom-restricted-width`. I finally set this in the style sheet:

~~~css
.custom-restricted-width {
    display: inline-block;
}
~~~


## Responsive vertical menu

I took this example from the provided [Pure.css] layouts on [Side Menu](https://purecss.io/layouts/side-menu/#home), with the associated 
[styles.css](https://github.com/pure-css/pure/blob/master/site/static/layouts/side-menu/styles.css) and 
[ui.js](https://github.com/pure-css/pure/blob/master/site/static/js/ui.js).

I had tried [ChatGPT](https://chatgpt.com) to generate the code, but it didn't work.

## Caveats

I noticed that the style sheet sets the width of the vertical menu to `150px`. Just changing the width is not so simple, because then suddenly the hamburger disappears. This restricts the length of the menu texts, but I ended up not changing this.
 
## Usage

Just download the three files from [github] and open `simple_index_vertical_menu.html` in your favorite browser.

## FontAwesome

[Font Awesome] is a good source for any kind of icons. However, they have hidden their CDN, since they prefer that you log in to get more information.

The CDN is here:

~~~html
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.6.0/css/all.min.css" integrity="sha512-Kc323vGBEqzTmouAECnVceyQqyqdsSiqLQISBL29aUW4U/M7pSPA/gEUZQqv1cwx4OnYxTxve5UMg5GT6L4JJg==" crossorigin="anonymous" referrerpolicy="no-referrer" />
~~~

And the free icons can be found here: [Font Awesome free icons](https://fontawesome.com/search?o=r&m=free).

[Pure.css]: https://purecss.io/
[github]: https://github.com/mmgreiner/vertical-responsive-menu
[Font Awesome]: https://fontawesome.com/