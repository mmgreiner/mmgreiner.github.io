---
title: Serving static pages locally
categories:
- programming
date: "2023-10-01T00:00:00Z"
tags:
- web
- jekyll
- hugo
- apache
- tomcat
- unix
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

I later moved all the [Jekyll] pages to [Hugo]. To publish and deploy the [Hugo] site locally, use the following command

~~~csh
hugo --destination ~/Sites/mypages --baseURL http://localhost/~mmgreiner/mypages
~~~

Make sure that the destination directory name corresponds to the last name of the `baseURL`, in this case `mypages-hugo`.

## Potential problems

If the server cannot be found, it may be, that apache is not running. Check with:

    apachectl

This may throw an error like this:

~~~
% apachectl
[Fri Dec 08 09:25:16.726003 2023] [so:error] [pid 18207] AH06663: Unable to find code signature authority 
on module at /usr/local/opt/php/lib/httpd/modules/libphp.so that matches authority name "Markus Greiner" configured on LoadModule directive.
httpd: Syntax error on line 192 of /private/etc/apache2/httpd.conf: 
Code signing error - not loading module at: /usr/local/opt/php/lib/httpd/modules/libphp.so
~~~

Basically, this may happen after you have upgraded `apachectl`. The code needs to be signed again, which is a complex procedure. 

From [stackoverflow](https://stackoverflow.com/questions/72787369/unable-to-code-sign-the-apache-modules-in-mac):

> To fix the above problem, I created Certificate Authority in mac machine by following - <https://www.simplified.guide/macos/keychain-ca-code-signing-create> and trying to codesign to sign the AEM Dispatcher module with below command

Only once, you have to create yourself a certificate. This is in detail described in <https://www.simplified.guide/macos/keychain-cert-code-signing-create>

Everytime `php` is updated, for instance through a `brew update` command, the `libphp.so` file needs to be signed again. 

- First, open *Keychain Access*, the *Keychains*, not the passwords. There, check that a certificate for *Markus Greiner* exists.
- Now in the terminal, do:

~~~
% codesign --sign "Markus Greiner" --force --keychain ~/Library/Keychains/login.keychain-db /usr/local/opt/php/lib/httpd/modules/libphp.so
~~~

Restart the server with:

    % sudo apachectl


[tomcat]: https://tomcat.apache.org
[Jekyll]: {{< param "jekyll_link" >}}
[Hugo]: {{< param "hugo_link" >}}
