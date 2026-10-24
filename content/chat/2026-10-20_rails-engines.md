---
title: Rails Engines
description: A concise introduction to Rails engines, including full vs. mountable
  engines, key files, mounting in a host app, routing, and testing.
categories:
- programming
tags:
- Rails
- engine
- mountable
- full
- routes
date: 2026-10-20
draft: false
showToc: false
---

# Rails engines

An engine is a miniature Rails application that you plug into a host app. It can have its own models, controllers, views, routes, migrations, assets, and tests. In fact, a Rails app *is* an engine: `Rails::Application` inherits from `Rails::Engine`. That's why everything you know about Rails carries over.

## Full vs. mountable engines

There are two flavors, created with a generator:

```bash
# Full engine: shares the host app's namespace
rails plugin new blog --full

# Mountable engine: isolated namespace (usually what you want)
rails plugin new blog --mountable
```

| | Full | Mountable |
|---|---|---|
| Namespace | Merges into the host app | Isolated (`Blog::Post`) |
| Routes | Merged into host's routes | Own routes, mounted at a path |
| Table names | Normal (`posts`) | Prefixed (`blog_posts`) |
| Best for | Sharing code *within* your app | Self-contained features or reusable gems |

If you're organizing a large app, go mountable. Full engines are rare.

## What the generator gives you

```
blog/
├── app/
│   ├── controllers/blog/
│   ├── models/blog/
│   ├── views/blog/
│   └── helpers/blog/
├── config/routes.rb
├── db/migrate/
├── lib/blog/engine.rb
├── lib/blog.rb
├── blog.gemspec
└── test/dummy/        # a throwaway Rails app used to test the engine
```

The key file is `lib/blog/engine.rb`:

```ruby
module Blog
  class Engine < ::Rails::Engine
    isolate_namespace Blog
  end
end
```

`isolate_namespace` is what makes it mountable. It scopes controllers, models, routes, and table names under `Blog`, so nothing collides with the host app.

## Using it in a host app

**1. Add it to the Gemfile** (a local path works great for an in-repo engine):

```ruby
gem "blog", path: "engines/blog"
```

**2. Mount it in `config/routes.rb`:**

```ruby
Rails.application.routes.draw do
  mount Blog::Engine, at: "/blog"
end
```

**3. Install its migrations:**

```bash
rails blog:install:migrations
rails db:migrate
```

Alternatively, you can skip the copying and have the host load the engine's migrations directly:

```ruby
# lib/blog/engine.rb
initializer "blog.migrations" do |app|
  unless app.root.to_s == root.to_s
    app.config.paths["db/migrate"] << root.join("db/migrate").to_s
  end
end
```

That's convenient for in-repo engines, since you don't end up with duplicate migration files.

## Routing between engine and host

Because the engine is isolated, route helpers are separate:

```ruby
# Inside the engine, linking to an engine route
blog_post_path(post)

# Inside the engine, linking to a HOST route
main_app.root_path

# Inside the host app, linking to an ENGINE route
blog.posts_path
```

Forgetting `main_app.` inside engine views is the most common beginner error.

## Talking to the host app

Engines often need to know about the host's `User` model or authentication. Don't hard-code it. Use configuration:

```ruby
# engines/blog/lib/blog.rb
module Blog
  mattr_accessor :author_class
  self.author_class = "User"
end
```

```ruby
# host app: config/initializers/blog.rb
Blog.author_class = "Account"
```

```ruby
# engines/blog/app/models/blog/post.rb
module Blog
  class Post < ApplicationRecord
    belongs_to :author, class_name: Blog.author_class
  end
end
```

For authentication, a common pattern is inheriting from a host-configurable controller:

```ruby
module Blog
  class ApplicationController < ::ApplicationController
    # now inherits the host's current_user, before_actions, layout, etc.
  end
end
```

## Overriding engine pieces from the host

Hosts can override views by putting a file at the same path in the host app:

```
# host app
app/views/blog/posts/index.html.erb   # overrides the engine's version
```

Models and controllers can be extended with `class_eval` or `Module#prepend` in an initializer or `to_prepare` block:

```ruby
# host app: config/initializers/blog.rb
Rails.application.config.to_prepare do
  Blog::Post.class_eval do
    scope :featured, -> { where(featured: true) }
  end
end
```

## Testing

The engine has its own test suite, running against `test/dummy`, a small Rails app that mounts the engine:

```bash
cd engines/blog
bin/rails test
```

This is a big benefit: you can test the engine in isolation without booting your whole host app.

## When engines are a good fit

- You want hard-ish boundaries between parts of a large app (billing, admin, blog, etc.) while keeping one deploy
- You're building something reusable across several Rails apps (publish it as a gem)
- You want separately testable, separately ownable modules

## Downsides to know about

- **Boilerplate**: more files, a gemspec, a dummy app, namespaced everything
- **Migrations**: the copy-vs-load decision adds friction
- **Boundaries are soft**: engines can still reach into each other's models; nothing enforces separation (that's where Packwerk helps)
- **Coupling to the host**: engines that need the host's `User` or auth get awkward without careful configuration
- **Overkill for small apps**: if you just want tidy folders, plain namespaces or Packwerk are lighter

## Quick recipe for an in-repo engine

```bash
mkdir engines
rails plugin new engines/billing --mountable --skip-git
```

```ruby
# Gemfile
gem "billing", path: "engines/billing"

# config/routes.rb
mount Billing::Engine, at: "/billing"
```

Then generate things inside it as usual:

```bash
cd engines/billing
bin/rails g scaffold invoice number:string total:decimal
```

If you tell me what the engines would hold (e.g., distinct domains in a big app vs. a reusable gem), I can suggest how to structure the boundaries and the host integration.

🤖 created with help of ai

# Moving an existing Rails app into an engine

There are two common goals, and the steps differ a bit:

- **A. Wrap the whole app** so it lives as an engine inside a new top-level host (so you can later add sibling engines).
- **B. Extract part of the app** (e.g., billing) into an engine and keep the rest as the host.

The mechanics are the same either way: generate an engine skeleton, move files in, namespace them, and fix up routes, migrations, and tests. Do it in small steps on a branch, keeping the test suite green.

## Before you start

- Make sure you have decent test coverage. Namespacing changes break things in surprising places.
- Commit everything, work on a branch.
- Decide the engine name and namespace (e.g., `Blog`, `Billing`). You'll need to live with it.

## Step 1: Generate the engine skeleton

```bash
mkdir engines
rails plugin new engines/billing --mountable --skip-git
```

Add it to the host Gemfile and mount it:

```ruby
# Gemfile
gem "billing", path: "engines/billing"

# config/routes.rb
mount Billing::Engine, at: "/billing"
```

Run `bundle install`. At this point the engine exists but is empty.

## Step 2: Move the code and namespace it

Move the relevant files into the engine, adding the namespace in both the **directory** and the **module**:

```
app/models/invoice.rb           →  engines/billing/app/models/billing/invoice.rb
app/controllers/invoices_controller.rb
                                →  engines/billing/app/controllers/billing/invoices_controller.rb
app/views/invoices/             →  engines/billing/app/views/billing/invoices/
app/helpers/invoices_helper.rb  →  engines/billing/app/helpers/billing/invoices_helper.rb
```

Then wrap the classes:

```ruby
# before
class Invoice < ApplicationRecord
end

# after
module Billing
  class Invoice < ApplicationRecord
  end
end
```

Things that need updating as you go:

- **Controllers** should inherit from `Billing::ApplicationController` (which you can point at the host's `::ApplicationController` if you want shared auth and layout).
- **Views**: partial paths and `render` calls need the namespace (`billing/invoices/form`).
- **Routes**: move the relevant routes into `engines/billing/config/routes.rb`:

  ```ruby
  Billing::Engine.routes.draw do
    resources :invoices
  end
  ```

- **Route helpers**: inside the engine, host routes need `main_app.`; from the host, engine routes need `billing.`.
- **Other code** that references `Invoice` now needs `Billing::Invoice`: other models, jobs, mailers, services, serializers, specs, and factories. Grep for these.
- **Other Ruby code**: move `lib/`, jobs, mailers, concerns, and service objects the engine depends on, under the same namespace.

## Step 3: Handle the database (the tricky part)

`isolate_namespace` makes table names default to `billing_invoices`, but your existing table is `invoices`. You have two options:

**Option 1: Keep the existing table names (least risky).** Declare them explicitly:

```ruby
module Billing
  class Invoice < ApplicationRecord
    self.table_name = "invoices"
  end
end
```

Or, for the whole engine, override the prefix:

```ruby
module Billing
  def self.table_name_prefix
    ""
  end
end
```

No schema change, no data migration. This is usually the right call for an existing production app.

**Option 2: Rename tables to the engine prefix.** Cleaner long-term, but it's a real migration with a deploy risk:

```ruby
class RenameInvoicesToBillingInvoices < ActiveRecord::Migration[7.1]
  def change
    rename_table :invoices, :billing_invoices
  end
end
```

If you do this, also check foreign keys, indexes, polymorphic `*_type` columns (which store class names like `"Invoice"` and would need updating to `"Billing::Invoice"`), and any raw SQL.

**Migrations files:** Your existing migrations already ran, so don't move them blindly. Either leave the old ones in the host, or move them and make sure the engine's migration path is loaded by the host (see the `initializer` snippet from earlier). Whatever you do, don't let the same migration version get copied twice.

## Step 4: Move the tests

- Move specs/tests next to the code in the engine.
- Fix factories and fixtures for the new class names.
- Configure the engine's `test/dummy` (or spec setup) so it boots and can run in isolation.
- Make sure the **host** suite still passes too, since it now depends on the engine.

## Step 5: Dependencies and the host boundary

Move gems that only the engine uses into the engine's `.gemspec`:

```ruby
# engines/billing/billing.gemspec
spec.add_dependency "rails", ">= 7.1"
spec.add_dependency "stripe"
```

For things the engine needs from the host (e.g., `User`, `current_user`), don't reference them directly. Use a config setting:

```ruby
module Billing
  mattr_accessor :user_class
  self.user_class = "User"
end
```

Then `belongs_to :user, class_name: Billing.user_class`.

## Step 6: Move incrementally

Don't try to do everything at once. A safe order:

1. Create and mount the empty engine. Deploy it.
2. Move one model and its tests. Fix references. Green suite.
3. Move its controller, views, and routes.
4. Repeat for the next piece.
5. Clean up: remove dead routes, unused gems, old directories.

Redirects from the old URL paths to the new mount point help if users or other systems have bookmarked the old URLs:

```ruby
# host routes.rb
get "/invoices", to: redirect("/billing/invoices")
```

## If you want to wrap the *entire* app (Goal A)

The approach is the same but bigger:

1. Create a new, thin host app (just routes, auth, config).
2. Create one engine (e.g., `Core` or the app's name) and move all of `app/`, `lib/`, and relevant `config/` into it with a namespace.
3. Mount it at `/`: `mount Core::Engine, at: "/"`.
4. Gradually split `Core` into smaller engines over time.

This is a large change for little immediate benefit; it's usually better to extract focused pieces (Goal B) rather than the whole app in one go.

## Common gotchas

- **Autoloading errors** after namespacing: the file path must match the constant (`billing/invoice.rb` defines `Billing::Invoice`). Run `bin/rails zeitwerk:check`.
- **`ApplicationRecord` / `ApplicationController` confusion**: inside the engine, an unqualified `ApplicationRecord` resolves to `Billing::ApplicationRecord`. Decide whether it should inherit from the host's, and reference `::ApplicationRecord` when you mean the host's.
- **Polymorphic associations and STI**: stored class names change when you add a namespace. Plan a data migration.
- **Background jobs**: queued jobs serialized with the old class name (`InvoiceJob`) will fail after the rename. Drain the queue or keep an alias for a while.
- **Route helper breakage** in mailers and views (`main_app.` / engine proxy).
- **Cached/serialized class names** in Redis, `ActiveStorage` attachments (`record_type`), `ActionText`, and audit logs.
- **Asset pipeline**: engine assets must be added to the host's precompile list or manifest.

## Rough checklist

- [ ] Branch + green tests
- [ ] Generate mountable engine, add to Gemfile, mount
- [ ] Move one slice of code, add namespace, fix references
- [ ] Decide table-name strategy (keep names vs. rename)
- [ ] Move routes and fix helpers (`main_app.` / engine proxy)
- [ ] Move tests; engine and host suites both green
- [ ] Move gem dependencies to the gemspec
- [ ] `zeitwerk:check`
- [ ] Handle polymorphic/STI/job queue class-name changes
- [ ] Deploy incrementally; repeat for the next slice

If you tell me the size of the app and which part you want to move first (or whether you want to wrap the whole thing), I can walk through a more specific plan, including how to handle your models' associations across the new boundary.