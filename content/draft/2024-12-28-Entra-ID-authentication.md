---
title: Authentication with Entra-ID
categories:
- programming
date: 2024-12-28
draft: true
tags:
- Sinatra
- ruby
- Azure
- Authentication
showToc: true
---

## Azure

First, set up in Azure. see: 

(taken from ChatGPT):
Make sure you’ve already registered your application in the Azure portal:

* Go to the **Azure portal** > **Azure Active Directory** > **App registrations** > **New registration**.
* Set the **redirect URI** to something like `http://localhost:4567/auth/azure_oauth2/callback`.
* Take note of:
  * **Application (Client) ID**
  * **Directory (Tenant) ID**
  * **Client Secret** (create one under **Certificates & secrets**)
  * 

Also check https://blog.corsego.com/microsoft-azure-omniauth

## Omniauth

https://github.com/RIPAGlobal/omniauth-entra-id?tab=readme-ov-file

## With Sinatra

Set up Sinatra routes
https://pavitkaur05.github.io/post/omniauth-with-sinatra/

## With Rails

## Code

see /Users/mmgreiner/Projects/ruby/sinatra-auth/app.rb
