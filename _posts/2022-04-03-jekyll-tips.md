---
title:  "Jekyll Tips & Tricks"
categories: publishing
tags: search jekyll
---

This post covers some more Jekyll tips and tricks like searching a web page.

## Searching a Jekyll page

Web pages based on databases such as [WordPress][wordpress] search the data in the underlying database. Static web sites based on [Jekyll]({{site.jekyll_link}}) however don't have this database but only files.  

Good descriptions of how they can be searched:

- [Vanilla js search](https://gomakethings.com/how-to-create-a-vanilla-js-search-page-for-a-static-website/)


[wordpress]:    https://wordpress.com

## Copy to Clipboard button

See [Add copy to clipboard in Jekyll](https://www.aleksandrhovhannisyan.com/blog/how-to-add-a-copy-to-clipboard-button-to-your-jekyll-blog/)

## Link on same page

See this [StackOverflow](https://stackoverflow.com/questions/4629675/jekyll-markdown-internal-links) question. 

You have to enable `auto_ids` in `_config.yml`:

    kramdown:
        auto_ids: true

Then headlines have the id of the hyphon concatenated words. This chapter for example: `link-on-same-page`. Link to it using `#link-on-same-page`.

