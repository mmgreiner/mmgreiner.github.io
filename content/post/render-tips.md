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
---

# Deploying with SQlite

*Note*: adopted from ChatGPT.

By default, **SQLite is just a file on disk**, and on Render that disk is **ephemeral** unless you explicitly attach persistent storage. That’s why your DB disappears on deploy.

You have two proper options:

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

---

## ⚠️ If You Really Want to Keep SQLite

You must attach a **Persistent Disk** in Render.

### Steps:

1. In Render dashboard → Your Web Service
2. Go to **Disks**
3. Add a new disk
4. Mount it to something like:

```
/var/data
```

5. Change your `database.yml` production config:

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

## 🎯 What I Recommend

If this app:

* Has users
* Will grow
* Is more than a demo
* Needs reliability

→ Use PostgreSQL.

If this is:

* A personal side project
* A small internal tool
* A prototype

→ Persistent disk + SQLite is acceptable.


