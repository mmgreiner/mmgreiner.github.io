---
title: Ruby on Rails ticks and trips
description: Ruby on Rails with bulma, haml, etc.
date: 2024-11-12
draft: false
categories:
- programming
tags:
- ruby
- rails
- haml
- bulma
- simple.css
- slim
- "css framework"
showToc: true
---

I have been using [Ruby on Rails][rails] quite a lot lately due to its extremely rapid way to build proof of concepts for web applications.

This post collects some of my experience, also using diffeent templating engines and css frameworks.

## Dotenv

Often it is the case that you rely on some parameters that are set in environment variables. This allows you to use different configuration during development, testing, and production.

The gem [dotenv-rails](https://github.com/bkeepers/dotenv) is ideal for this. First, add it to the Gemfile:

  % bundle add dotenv-rails --group=development,test

Now create a `.env` file in the root directory. Make sure that this file is ignored by git, since you may keep confidential keys here.

~~~~text
MY_URL=http://localhost:3000
~~~~ 

Inside your rails application, you can now access the value like this:

~~~~ruby
my_url = ENV["MY_URL"]
# or
my_url = ENV.fetch "MY_URL"
~~~~

`fetch` throws an error, if the value is not defined, which may be helpful if you are unsure about the environment.

Dotenv should automatically load when the Rails app boots. If this doesn't work, load it inside the configuration:

~~~ruby
# config/application.rb

# Load .env file in development and test environments
if Rails.env.development? || Rails.env.test?
  Dotenv::Railtie.load
end
~~~

**Note 1**: in rake tasks, you may have to include:

~~~~ruby
require "dotenv/load"
~~~~

**Note 2**: if you want to test production environment on your local machine before deployment, you can load the environment variables like this:

    dotenv -f .env rails server



## Templating

Template engines are used to create the html for the views. The standard rails templating engine in [ERB](https://guides.rubyonrails.org/action_view_overview.html#templates), the **E**mbedded **R**u**B**y templating engine. However, there are other templating engines which are more concise. 

### Haml

[haml](https://haml.info/) is a templating engine with the goal:

> Haml (HTML abstraction markup language) is based on one primary principle: markup should be beautiful. 

It replaces HTML templates like:

```html
<section class="container">
  <h1><%= post.title %></h1>
  <h2><%= post.subtitle %></h2>
  <div class="content">
    <%= post.content %>
  </div>
</section>
```

with 

```haml
%section.container
  %h1= post.title
  %h2= post.subtitle
  .content
    = post.content
```

The [haml] gem is quite well documented. To use it in rails, first, install it and the helper tool:

    % bundle add haml-rails
    % bundle add html2haml

Then, you can convert the exising `.html.erb` files to `.html.haml` files:

    $ rails generate haml:application_layout convert

Finally, you have to remove the `application.html.erb` file. Or, you can convert all your *erb* files to *haml*:

    $ rails haml:erb2haml

This will also ask you if you want to delete all the *erb* files.

### Slim

After [haml] I moved on to using [slim], which is even more concise. You can add it to the Gemfile `bundle add slim-rails`, and then start using it. Basically, it is like haml without the `%` for html tags.

To always use it when generating views, create the project as follows:

```
% rails new my_project --template-engine=slim
```

For an existing rails application, you can change the configuration in `config/application.rb`:

```ruby
module MyProject
  class Application < Rails::Application
    config.generators.template_engine = :slim
    # Other configuration settings...
  end
end
```

[slim]: https://slim-template.github.io/


### Markdown

Sometimes, particularely if you want to view larger texts, you may have the need for markdown rendering.

I have found this [article](https://www.writesoftwarewell.com/how-to-render-markdown-views-in-rails/) which explains the steps.

1. Install the gem `bundle add  redcarpet`.

2. Add an initializer `config/initializers/markdown.rb`

~~~ruby
require "redcarpet"

ActionView::Template.register_template_handler :md, lambda { |template, source|
  renderer = Redcarpet::Render::HTML.new
  markdown = Redcarpet::Markdown.new(renderer, extensions = {})
  "#{markdown.render(source).inspect}.html_safe"
}
~~~

3. Restart the server `rails server`.

4. Write Views with `.html.md` extensions. They will automatically be rendered. 

## CSS Frameworks

### Bulma

I picked [bulma] as the css framework after many years of working with [purecss](https://pure-css.github.io/) mainly becouse [bulma] offered better responsive designs, particularly when it came to responsive navigation menus.

To use [bulma] with [rails], you have to do the following modifications:

Add the stylesheet to the  header:

```haml
%html
  %head
    %meta{:content => "text/html; charset=UTF-8", "http-equiv" => "Content-Type"}/
    ...
    = stylesheet_link_tag "https://cdn.jsdelivr.net/npm/bulma@1.0.2/css/bulma.min.css"
```

Then use `container` and apply it to all the elements of the body:

```haml
%body
%section.section        # added
  .container            # added
    = yield
```

That's it. Now you have the default fonts and colors of [bulma].

Obviously, if you want to make forms look nicer you have to change the auto-generated erb / haml files. A good guide is at the [bulma forms](https://bulma.io/documentation/form/general/) documentation.

[rails]: https://guides.rubyonrails.org/index.html
[haml]: https://github.com/haml/haml-rails
[bulma]: https://bulma.io/

This gives some tips and trick for Ruby on Rails.

### Simple.css

After Bulma, I moved to [Simple.css]. For more information, see the [post on Simple.css]({{< ref "2025-01-01-simplecss" >}}).

[Simple.css]: https://simplecss.org/

## Mimimum application

Create a minimum application which uses *sqlite3* as database (default since rails 8.0) and [slim] as templating engine and without javascript, mailboxes, etc.

```
rails new my-app -M -C -A -J --template-engine=slim
```

## Enums

There are many sites explaining how to use [enums in rails](), but all of them describe the use case where the enum is stored as an integer in the database. 

My much more frequent use case is using strings in the database. I found this on [stackoverflow](https://stackoverflow.com/questions/32938729/how-to-store-enum-as-string-to-database-in-rails) which gives a good way to do it.

```ruby
class Work < ApplicationRecord
  enum status: %i[canceled offering running payment rating done].index_with(&:to_s)
end
```

The method `index_with` is part of `enum` and converts an enum into a hash. You can check this:

~~~ruby
Person.statuses
=> 
{"canceled" => "canceled",
 "offering" => "offerng",
 "running" => "running",
 "payment" => "payment",
 "rating" => "rating",
 "done" => "done"}
~~~

Then, in the view, you can use:

```ruby
form.select :status, Work::statuses
```

### Multi-value enums

If you need enums which can contain multiple values, you should not use `enum` but work with stored arrays and `serialize`.

```ruby
serialize :categories, coder: JSON
CATEGORIES = %w[news sports entertainment politics technology]
validate :categories_must_be_valid

private

def categories_must_be_valid
  puts "inside validator"
  invalid_categories = categories - CATEGORIES
  errors.add(:categories, "contain invalid categories: #{invalid_categories.join(', ')}") if invalid_categories.any?
end
```

You can now assign the values:

```ruby
article.categories = [ "news", "sports" ]
article.categories << "politics"
```

In the view, you would use:

~~~html
= form.collection_check_boxes :categories, XY::CATEGORIES.map{ [_1, _1]}, :first, :first, include_hidden: false
~~~

The `include_hidden: false` parameter is needed, otherwise the returned parameters would contains an empty `""` first value which would break validation.


## Various tricks

Hit <http://localhost:3000/rails/info/routes> to get a list of all routes.

## Database views

Often, database views are an easy way to increase performance. This is how you create views.

### Create Migration

Create a migration `rails g migration CreateView` and change it:

```ruby
class CreateMyView < ActiveRecord::Migration[6.1]
  def up
    execute <<-SQL
      CREATE VIEW my_view AS
      SELECT column1, column2, column3
      FROM table1
      JOIN table2 ON table1.id = table2.table1_id
      WHERE condition1 AND condition2
    SQL
  end

  def down
    execute <<-SQL
      DROP VIEW my_view
    SQL
  end
end
```

and then run the migration `rails db:migrate`. 

### Create model

The model will look something like this:

```ruby
class MyView < ApplicationRecord
  self.table_name = 'my_view'
  self.inheritance_column = nil   # necessary for Single-table-inheritance STI
  def readonly?
    true
  end
end
```

### Create controller and views

Now you have to create the controller and the views, but not the model. 

```
rails g scaffold_controller MyView
```

## Useful things with the console

I found this helpful article [Useful thinkgs you can do with the rails console](https://impactahead.com/dev/useful-things-you-can-do-with-rails-console). 

## Quick reference

I found an excellent quick reference guide to Ruby on [zenspider][quickref].

## More complex associations

I asked this to chatGPT:

> Using ruby active record, how can I model the following situation:
> An order has a supplier of class BusinessPartner, and a producer also of class BusinessPartner. How is the ruby code for the model and for the database migrations

I got the following response

To model this scenario in Ruby on Rails using Active Record, where an `Order` has both a `supplier` and a `producer`, each of which belongs to the same class `BusinessPartner`, you can use the concept of **self-referential associations** with polymorphic-like relationships. This allows you to associate the `BusinessPartner` model in two different roles within the `Order` model.

### Step 1: Database Migrations

You'll need to set up your database tables to accommodate these associations. Here's how you can create the necessary migrations.

#### 1.1 Create the `BusinessPartner` model and migration:

```bash
rails generate model BusinessPartner name:string
```

This will generate a migration file. Make sure it looks like this:

```ruby
# db/migrate/20241113000001_create_business_partners.rb
class CreateBusinessPartners < ActiveRecord::Migration[7.0]
  def change
    create_table :business_partners do |t|
      t.string :name, null: false
      t.timestamps
    end
  end
end
```

#### 1.2 Create the `Order` model and migration:

```bash
rails generate model Order supplier_id:integer producer_id:integer
```

This will generate another migration file. Update it as follows:

```ruby
# db/migrate/20241113000002_create_orders.rb
class CreateOrders < ActiveRecord::Migration[7.0]
  def change
    create_table :orders do |t|
      t.references :supplier, null: false, foreign_key: { to_table: :business_partners }
      t.references :producer, null: false, foreign_key: { to_table: :business_partners }
      t.timestamps
    end
  end
end
```

#### 1.3 Run the Migrations

Run the migrations to create the tables:

```bash
rails db:migrate
```

### Step 2: Model Setup

Now, let's set up the Ruby models to reflect this relationship.

#### 2.1 BusinessPartner Model

```ruby
# app/models/business_partner.rb
class BusinessPartner < ApplicationRecord
  has_many :supplied_orders, class_name: 'Order', foreign_key: 'supplier_id'
  has_many :produced_orders, class_name: 'Order', foreign_key: 'producer_id'
end
```

#### 2.2 Order Model

```ruby
# app/models/order.rb
class Order < ApplicationRecord
  belongs_to :supplier, class_name: 'BusinessPartner'
  belongs_to :producer, class_name: 'BusinessPartner'
end
```

### Step 3: Using the Models

Now that the models and migrations are set up, you can create `BusinessPartner` records and associate them with `Order` records:

#### 3.1 Creating Business Partners

```ruby
supplier = BusinessPartner.create(name: 'Supplier Co.')
producer = BusinessPartner.create(name: 'Producer Ltd.')
```

#### 3.2 Creating an Order

```ruby
order = Order.create(supplier: supplier, producer: producer)
```

#### 3.3 Querying the Data

```ruby
# Fetch the supplier for an order
order.supplier

# Fetch the producer for an order
order.producer

# List all orders supplied by a specific business partner
supplier.supplied_orders

# List all orders produced by a specific business partner
producer.produced_orders
```

### Explanation

- The `Order` model has two foreign keys, `supplier_id` and `producer_id`, both referencing the `BusinessPartner` model.
- The `belongs_to` association in the `Order` model specifies `class_name: 'BusinessPartner'` to indicate that these associations point to the `BusinessPartner` model.
- The `BusinessPartner` model includes `has_many` associations with customized foreign keys, allowing you to fetch orders where the partner is either a supplier or a producer.

This approach is clean and efficient, leveraging Rails' built-in association features without requiring polymorphic associations since both `supplier` and `producer` are of the same type (`BusinessPartner`).

## ActiveRecords associations

I have for the longest time been confused about how the associations are mapped to the database tables. 

A fairly good documentation can be found in the [ActiveRecord documentation](https://apidock.com/rails/ActiveRecord/Associations/ClassMethods).

For my own help I created this table, the shows how the migrations, the generated models, and the sql statements fit together.

<table>
<thead>
  <tr>
    <th></th>
    <th>Model A</th>
    <th>Model B</th>
  </tr>
</thead>
<tbody>
<tr>
  <td>Migration</td>
  <td>

```
bin/rails g model A title:string
```

</td>
  <td>

```
bin/rails g model B title:string a:references
```

</td>
<tr>
  <td>Migration code</td>
  <td>

```ruby
class CreateAs < ActiveRecord::Migration[7.1]
  def change
    create_table :as do |t|
      t.string :title

      t.timestamps
    end
  end
end
```

</td>
  <td>

```ruby
class CreateBs < ActiveRecord::Migration[7.1]
  def change
    create_table :bs do |t|
      t.string :title
      t.references :a, null: false, foreign_key: true

      t.timestamps
    end
  end
end
```

</td>

<tr>
  <td>Models</td>
  <td>

```ruby
class A < ApplicationRecord
    has_one :b
end
```

</td>
  <td>

```ruby
class B < ApplicationRecord
  belongs_to :a 
end
```

</td>
</tr>

<tr>
<td>Schema</td>
<td>

```ruby
create_table "as", force: :cascade do |t|
    t.string "title"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
end
```

</td>
<td>

```ruby
create_table "bs", force: :cascade do |t|
    t.string "title"
    t.integer "a_id", null: false
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.index ["a_id"], name: "index_bs_on_a_id"
end

add_foreign_key "bs", "as"
```

</td>

</tr>

<tr>
<td>Database</td>
<td>

```sql
CREATE TABLE IF NOT EXISTS "as" 
("id" integer PRIMARY KEY AUTOINCREMENT NOT NULL, 
"title" varchar, "created_at" datetime(6) NOT NULL,
"updated_at" datetime(6) NOT NULL);
```

</td>
<td>

```sql
CREATE TABLE IF NOT EXISTS "bs" 
("id" integer PRIMARY KEY AUTOINCREMENT NOT NULL,
"title" varchar, "a_id" integer NOT NULL, 
"created_at" datetime(6) NOT NULL, 
"updated_at" datetime(6) NOT NULL, 
CONSTRAINT "fk_rails_ddf8c0c4b5"
FOREIGN KEY ("a_id")
  REFERENCES "as" ("id")
);
CREATE INDEX "index_bs_on_a_id" ON "bs" ("a_id");
```

The foreign key index is generated on the `belongs_to` class.

</td>
</tr>
<tr>
<td>Creating</td>
<td>

```ruby
a = A.new(title: 'hallo A')
a.save!
==> INSERT INTO "as" ("title", "created_at", "updated_at") 
VALUES (?, ?, ?) RETURNING "id"  [["title", "hallo A"], 
["created_at", "2024-03-16 08:20:14.170060"], 
["updated_at", "2024-03-16 08:20:14.170060"]]
```

</td>
<td>

```ruby
b = B.new(title: 'hallo B')
b.save!
==> `raise_validation_error': Validation failed: 
A must exist (ActiveRecord::RecordInvalid)
```

b must have an association to a when saving.

```ruby
b = B.new(title: 'hallo B', a: a)
b.save!
TRANSACTION (0.1ms)  begin transaction
B Create (2.9ms)  INSERT INTO "bs" 
("title", "a_id", "created_at", "updated_at") VALUES (?, ?, ?, ?) 
RETURNING "id"  [["title", "hallo B"], ["a_id", 4], 
["created_at", "2024-03-16 08:25:29.621566"], 
["updated_at", "2024-03-16 08:25:29.621566"]]
TRANSACTION (0.9ms)  commit transaction
```

This creates a and the associated b.

</td>
</tr>

<tr>
<td>Access</td>
<td>

Access to the reference via the foreign key:

```ruby
A.find(4).b
  A Load (0.2ms)  SELECT "as".* FROM "as" WHERE "as"."id" = ? 
  LIMIT ?  [["id", 4], ["LIMIT", 1]]
  B Load (0.2ms)  SELECT "bs".* FROM "bs" WHERE "bs"."a_id" = ? 
  LIMIT ?  [["a_id", 4], ["LIMIT", 1]]
```

</td>
<td>

Access with two sequential selects.

```ruby
B.find(3).a
  B Load (0.2ms)  SELECT "bs".* FROM "bs" WHERE "bs"."id" = ? 
  LIMIT ?  [["id", 3], ["LIMIT", 1]]
  A Load (0.1ms)  SELECT "as".* FROM "as" WHERE "as"."id" = ? 
  LIMIT ?  [["id", 4], ["LIMIT", 1]]
```

</td>
</tr>

<tr>
<td>what</td>
<td>

```ruby

```

</td>
<td>

```ruby

```

</td>
</tr>
</tbody>
</table>

## One-liners

A very interesting blog on using ruby one-lines, mainly with text files,
was found under [Ruby One.liners](https://learnbyexample.github.io/learn_ruby_oneliners/cover.html).

## Instance variables and class instance variables

This is totally confusing when coming from another programming language. I found a fairly good explanation of instance variables and class instance variables [here](https://fernandobasso.dev/ruby/classes-and-objects.html).

[quickref]: https://www.zenspider.com/ruby/quickref.html#types