---
title:  "Jekyll Tips & Tricks"
categories: publishing
tags: search jekyll
---

This post covers some more Jekyll tips and tricks like searching a web page.

## Searching a Jekyll page

Web pages based on databases such as [WordPress][wordpress] search the data in the underlying database. Static web sites based on [Jekyll] however don't have this database but only files.  

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


## Publish to OSX local apache server

OSX has a local apache server build in, see [Setting up a local web server](https://discussions.apple.com/docs/DOC-250006086).

The pages reside under `~<user>/Sites`. Normally, you would create a sub-site in that directory, for instance: `~<user>/mypages`.

To build your [Jekyll] static pages directly to that site, use (assuming the user is `mmgreiner`):

~~~~
jekyll build --drafts -d ~/Sites/mypages --baseurl "~mmgreiner/mypages"
~~~~

Note the quotes on the last argument. If you don't use these quotes, the shell will expand the `~` and it will pass the wrong base url.

You can now browse to <http://localhost/~mmgreiner/mypages> to see your [Jekyll] build local static pages.


[Jekyll]: {{site.jekyll_link}}

