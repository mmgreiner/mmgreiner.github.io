---
title: Render deployment tips
description: Various tips use to deploy Rails applications to render
date: 2026-02-12
draft: false
categories:
- programming
tags:
- ruby
- rails
- render
showToc: true
---


# Using private repository

If you want to connect a private repository to [render], proceed as follows:

- go to the [Render Dashboard](https://dashboard.render.com), on the top right click on `+ New` and select *Web Service*
- you are redirected to a list with a tab *Git Provider*
- select *Credentials (1)*
- Under *Select deployment credentials*, pick you github repository
- Now do *Configure in GitHub*
- You are redirected to *GitHub*. 
_ Scroll down, until you see *Repository Access*
- Pick *only select repositories*, and pick the one you need.

Voilà, you have now connected it to GitHub

[render]: https://dashboard.render.com


# Deploying with SQlite

*Note*: adopted from ChatGPT.

By default, **SQLite is just a file on disk**, and on Render that disk is **ephemeral** unless you explicitly attach persistent storage. That’s why your DB disappears on deploy.

You have two proper options:

- Really use SQLite
- Switch to PostgreSQL


## ⚠️ If You Really Want to Keep SQLite

You must attach a **Persistent Disk** in Render.

### Steps:

1. In Render dashboard → Your Web Service
2. Go to **Disks**
3. Add a new disk: **Note**: this is not available in the free of charge instances.
4. Mount it to something like:

```
/var/data
```

5. Change your `config/database.yml` production config:

```yaml
production:
  adapter: sqlite3
  database: /var/data/production.sqlite3
```

Now the SQLite file lives on persistent storage.

---

## 🚨 Important Limitations of SQLite on Render

Even with a disk:

* ❌ No horizontal scaling
* ❌ Risk of file locking issues
* ❌ Slower under concurrent load
* ❌ Not recommended for production apps with users

SQLite is single-writer.


---

## ✅ Recommended (Best Practice): Switch to PostgreSQL on Render

This is the **correct production solution**.

SQLite is fine for local development, but for production deployments (especially on platforms like Render), you should use a managed database.

### Why?

* Render services are ephemeral
* Deploys can recreate containers
* Horizontal scaling won’t work with SQLite
* Risk of data loss

### What to do:

1. Create a **PostgreSQL database** in Render

   * Render Dashboard → New → PostgreSQL

2. Add the `pg` gem:

```ruby
# Gemfile
gem "pg"
```

3. Update `config/database.yml`:

```yaml
production:
  adapter: postgresql
  encoding: unicode
  database: <%= ENV["DATABASE_NAME"] %>
  username: <%= ENV["DATABASE_USER"] %>
  password: <%= ENV["DATABASE_PASSWORD"] %>
  host: <%= ENV["DATABASE_HOST"] %>
  port: 5432
  pool: 5
```

4. Set environment variables in Render (they usually auto-provide them)

5. Run:

```
rails db:migrate
```

After that, your DB persists across deploys and restarts.



# Using authentication

Starting with rails version 8, you can very easily add authentication to a rails app (see [Rails security](https://guides.rubyonrails.org/security.html):

~~~csh
bin/rails generate authentication
bin/rails db:migrate
~~~

This works fine. You can then easily augment the navigation in the `app/views/layouts/application.html.erb` as follows:

~~~
<% if authenticated? %>
    <li><%= authenticated?.user.email_address%> </li>
    <li><%= button_to "Sign Out", session_path, method: :delete  %></li>
<% else %>
    <li><%= link_to "Sign In", new_session_path %></li>
<% end %>
~~~

It all works fine during development. But after deploying to `render`, you get strange errors. They were introduced, since authentication starts using the cache.

So you also have to change the cache settings, particular if you are using SQlite:

~~~ruby
# config/environments/production.rb
config.cache_store = :solid_cache_store
~~~

in the render configuration, ensure that you migrate all databases. Define the settings `pre-deployment command` to be: 

~~~
bundle exec rails db:migrate && bundle exec rails db:migrate:cache && bundle exec rails db:migrate:queue
~~~

Otherwise, the cache database will never be set up and the application will crash.




