---
categories:
- programming
date: "2023-10-01T00:00:00Z"
tags:
- web
- jekyll
- hugo
title: Serving static pages locally
showshowToc: true
---

## Running build-in tomcat server

It is possible to deploy [Jekyll] sites locally, because OSX has a build-in [Apache tomcat server][tomcat]. The local sites reside under `~/Sites`. However, the setup of this server often changes with new versions of OSX.

John does a great job on documenting how to do this on [Apple Community](https://discussions.apple.com/docs/DOC-3083). For the latest version Sonoma 14.0, these instructions can be seen [here](https://discussions.apple.com/docs/DOC-250007792).

If you follow these instructions, you should be able to run <http://localhost/>, and it should respond with:

    It works!

If you browse to <http://localhost/~mmgreiner/>, you should see the files:

    Index of ~/mmgreiner

## Deploying Jekyll static site locally

To deploy your [Jekyll] generated static sites locally, use

~~~csh
jekyll build --drafts -d ~/Sites/mypages --baseurl "~mmgreiner/mypages"
~~~

If you now browse to <http://localhost/~mmgreiner/mypages/>, you should see your [Jekyll] sites.

## Deploy Hugo locally

To publish and deploy the [Hugo] site locally, use the following command

~~~csh
hugo --destination ~/Sites/mypages-hugo --baseURL http://localhost/~mmgreiner/mypages-hugo
~~~

Make sure that the destination directory name corresponds to the last name of the `baseURL`, in this case `mypages-hugo`.


[tomcat]: https://tomcat.apache.org
[Jekyll]: {{< param "jekyll_link" >}}
[Hugo]: {{< param "hugo_link" >}}
