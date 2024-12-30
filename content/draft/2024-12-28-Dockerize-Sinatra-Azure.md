---
categories:
- programming
date: 2024-12-28
draft: true
tags:
- Docker
- Sinatra
- ruby
- Azure
title: Docker, Sinatra and Azure
description: Dockerize a Sinatra App and Deploy it to Azure
showToc: true
---

This describes the most simple way to create a Docker container of a [Sinatra][sinatra] app and deploying it to [Azure][azure].

## Sinatra

> Sinatra is a DSL for quickly creating web applications in Ruby with minimal effort

As a prerequisite, you must have [ruby][ruby] installed.

~~~bash
mkdir my-app
cd my-app
bundle init
bundle add sinatra rackup puma
touch myapp.rb
~~~

The `Gemfile` looks something like this:

~~~
# frozen_string_literal: true
source "https://rubygems.org"

gem "sinatra", "~> 4.1"
gem "rackup", "~> 2.2"
gem "puma", "~> 6.5"
~~~

We have a very simple ruby application in `app.rb`:

~~~ruby
require 'sinatra'
require 'logger'

set :logging, :debug

get '/' do
  logger.info "inside home"
  "Hallo World"
end
~~~

Run this little app:

~~~bash
% % ruby app.rb
== Sinatra (v4.1.1) has taken the stage on 4567 for development with backup from Puma
Puma starting in single mode...

* Puma version: 6.5.0 ("Sky's Version")
* Ruby version: ruby 3.4.1 (2024-12-25 revision 48d4efcb85) +PRISM [x86_64-darwin23]
* Min threads: 0
* Max threads: 5
* Environment: development
*       PID: 37241
* Listening on http://127.0.0.1:4567
* Listening on http://[::1]:4567
  Use Ctrl-C to stop
~~~

To use the [rake middleware](https://github.com/rack/rack), first create a rake file `config.ru`:

~~~~ruby
require './app'
run Sinatra::Application
~~~~

Then start the server:

~~~sh
% bundle exec rackup -p 9292
Puma starting in single mode...
* Puma version: 6.5.0 ("Sky's Version")
* Ruby version: ruby 3.4.1 (2024-12-25 revision 48d4efcb85) +PRISM [x86_64-darwin23]
*  Min threads: 0
*  Max threads: 5
*  Environment: development
*          PID: 61845
* Listening on http://127.0.0.1:9292
* Listening on http://[::1]:9292
Use Ctrl-C to stop
I, [2024-12-30T00:59:29.162530 #61845]  INFO -- : inside home
::1 - - [30/Dec/2024:00:59:29 +0100] "GET / HTTP/1.1" 200 11 0.0192
~~~

[sinatra]: https://sinatrarb.com/
[azure]: https://portal.azure.com/
[docker]: https://www.docker.com/
[ruby]: https://www.ruby-lang.org/en/

