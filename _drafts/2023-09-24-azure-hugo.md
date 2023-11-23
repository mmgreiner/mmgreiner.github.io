---
categories:
    - programming
    - web
tags:
    - hugo
    - render
    - azure
title: Static web app with authentication
toc: true
---

We will try to create a [Hugo] static web app that for some sub-pages uses authentication and authorization on azure to restrict access. The example builds on [publish hugo][azure-hugo].

## Hugo static web app

[Hugo] claims to be 

> The world’s fastest framework for building websites

It is based on [Go]({{site.go_link}}). We will create a new site, and will call it `country-road`, with the `ananke` theme:

~~~~
hugo new site country-road
cd country-road
git init
git branch -M main
git submodule add https://github.com/budparr/gohugo-theme-ananke.git themes/ananke
~~~~

I prefer yaml configuration to toml. `hugo.yaml`:

~~~~yaml
baseURL : http://example.org/
languageCode : en-us
title : "My New Hugo Site"
theme : ananke
~~~~

Test it:

~~~~
hugo server
~~~~

Commit the changes

~~~~
git add -A
git commit -m "initial commit"
~~~~

## Push to git

TBD

## Publish on Azure

Open Azure portal <https://portal.azure.com/#home>

Error: 

    No url found for submodule path 'hugo-static-app' in .gitmodules
    
For some reason, a subdirectory `hugo-static-app` was created, probably by git. removing this subdirectory solved this problem.

The deployment to azure also creates the file `.github/workflows/azure-static-web-apps-ashy-water-0f6c0b803.yml`.

Now you can look at the site at <https://ashy-water-0f6c0b803.3.azurestaticapps.net>

Good article on a very simple static web page with azure and authentication: <https://techcommunity.microsoft.com/t5/apps-on-azure-blog/how-to-setup-built-in-authentication-for-azure-static-web-apps/ba-p/3734709>

How to get the user name: <https://learn.microsoft.com/en-us/azure/static-web-apps/user-information?tabs=javascript>

Unfortunately, I never got Azure to run in the end, and I turned to other alternatives. 

## Publish to Render

I decided to publish the Hugo site to [Render], which turned out the be the easiest, even if authentication did not work.

The setup was straight forward is is excellently [documented](https://render.com/docs/deploy-hugo). 

[Render] also takes the pages directly from [Github]({{site.github_link}}), so the process is quite simple.


[Hugo]: https://gohugo.io
[azure-hugo]: https://learn.microsoft.com/en-us/azure/static-web-apps/publish-hugo
[host-azure]: https://gohugo.io/hosting-and-deployment/hosting-on-azure-static-web-apps/

[authentication]: https://learn.microsoft.com/en-us/azure/static-web-apps/authentication-authorization
[Render]: https://dashboard.render.com/

