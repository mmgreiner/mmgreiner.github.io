---
title: Ruby on Rails ticks and trips
description: Ruby on Rails with bulma, haml, etc.
date: 2024-11-12
categories:
- programming
tags:
- ruby
- rails
- haml
- bulma
showToc: true
---

I have been using [Ruby on Rails][rails] quite a lot lately due to its extremely rapid way to build proof of concepts for web applications.

However, I'm typically using [haml] for the view templates, and [bulma] as css framework.

## Haml

[haml](https://haml.info/) is a templating engine with the goal:

> Haml (HTML abstraction markup language) is based on one primary principle: markup should be beautiful. 

It replaces HTML templates like:

~~~~html
<section class="container">
  <h1><%= post.title %></h1>
  <h2><%= post.subtitle %></h2>
  <div class="content">
    <%= post.content %>
  </div>
</section>
~~~~

with 

~~~~haml
%section.container
  %h1= post.title
  %h2= post.subtitle
  .content
    = post.content
~~~~

The [haml] gem is quite well documented. To use it in rails, first, install it and the helper tool:

    % bundle add haml-rails
    % bundle add html2haml

Then, you can convert the exising `.html.erb` files to `.html.haml` files:

    $ rails generate haml:application_layout convert

Finally, you have to remove the `application.html.erb` file. Or, you can convert all your *erb* files to *haml*:

    $ rails haml:erb2haml

This will also ask you if you want to delete all the *erb* files.


## Bulma

I picked [bulma] as the css framework after many years of working with [purecss](https://pure-css.github.io/) mainly becouse [bulma] offered better responsive designs, particularly when it came to responsive navigation menus.

To use [bulma] with [rails], you have to do the following modifications:

Add the stylesheet to the  header:

~~~~haml
%html
  %head
    %meta{:content => "text/html; charset=UTF-8", "http-equiv" => "Content-Type"}/
    ...
    = stylesheet_link_tag "https://cdn.jsdelivr.net/npm/bulma@1.0.2/css/bulma.min.css"
~~~~

Then use `container` and apply it to all the elements of the body:

~~~~haml
%body
%section.section        # added
  .container            # added
    = yield
~~~~

That's it. Now you have the default fonts and colors of [bulma].

Obviously, if you want to make forms look nicer you have to change the auto-generated erb / haml files. A good guide is at the [bulma forms](https://bulma.io/documentation/form/general/) documentation.


[rails]: https://guides.rubyonrails.org/index.html
[haml]: https://github.com/haml/haml-rails
[bulma]: https://bulma.io/



This gives some tips and trick for Ruby on Rails.

## Quick reference

I found an excellent quick reference guide to Ruby on [zenspider][quickref].


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

~~~
bin/rails g model A title:string
~~~
  </td>
  <td>
  
~~~
bin/rails g model B title:string a:references
~~~

  </td>
<tr>
  <td>Migration code</td>
  <td>

~~~ruby
class CreateAs < ActiveRecord::Migration[7.1]
  def change
    create_table :as do |t|
      t.string :title

      t.timestamps
    end
  end
end
~~~
  </td>
  <td>

~~~ruby
class CreateBs < ActiveRecord::Migration[7.1]
  def change
    create_table :bs do |t|
      t.string :title
      t.references :a, null: false, foreign_key: true

      t.timestamps
    end
  end
end
~~~
  </td>

<tr>
  <td>Models</td>
  <td>

~~~ruby
class A < ApplicationRecord
    has_one :b
end
~~~
  </td>
  <td>

~~~ruby
class B < ApplicationRecord
  belongs_to :a 
end
~~~
  </td>
</tr>

<tr>
<td>Schema</td>
<td>

~~~ruby
create_table "as", force: :cascade do |t|
    t.string "title"
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
end
~~~
</td>
<td>

~~~ruby
create_table "bs", force: :cascade do |t|
    t.string "title"
    t.integer "a_id", null: false
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.index ["a_id"], name: "index_bs_on_a_id"
end

add_foreign_key "bs", "as"
~~~
</td>

</tr>

<tr>
<td>Database</td>
<td>

~~~sql
CREATE TABLE IF NOT EXISTS "as" 
("id" integer PRIMARY KEY AUTOINCREMENT NOT NULL, 
"title" varchar, "created_at" datetime(6) NOT NULL,
"updated_at" datetime(6) NOT NULL);
~~~
</td>
<td>

~~~sql
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
~~~

The foreign key index is generated on the `belongs_to` class.

</td>
</tr>
<tr>
<td>Creating</td>
<td>

~~~ruby
a = A.new(title: 'hallo A')
a.save!
==> INSERT INTO "as" ("title", "created_at", "updated_at") 
VALUES (?, ?, ?) RETURNING "id"  [["title", "hallo A"], 
["created_at", "2024-03-16 08:20:14.170060"], 
["updated_at", "2024-03-16 08:20:14.170060"]]
~~~
</td>
<td>

~~~ruby
b = B.new(title: 'hallo B')
b.save!
==> `raise_validation_error': Validation failed: 
A must exist (ActiveRecord::RecordInvalid)
~~~

b must have an association to a when saving.

~~~ruby
b = B.new(title: 'hallo B', a: a)
b.save!
TRANSACTION (0.1ms)  begin transaction
B Create (2.9ms)  INSERT INTO "bs" 
("title", "a_id", "created_at", "updated_at") VALUES (?, ?, ?, ?) 
RETURNING "id"  [["title", "hallo B"], ["a_id", 4], 
["created_at", "2024-03-16 08:25:29.621566"], 
["updated_at", "2024-03-16 08:25:29.621566"]]
TRANSACTION (0.9ms)  commit transaction
~~~

This creates a and the associated b.
</td>
</tr>

<tr>
<td>Access</td>
<td>

Access to the reference via the foreign key:

~~~ruby
A.find(4).b
  A Load (0.2ms)  SELECT "as".* FROM "as" WHERE "as"."id" = ? 
  LIMIT ?  [["id", 4], ["LIMIT", 1]]
  B Load (0.2ms)  SELECT "bs".* FROM "bs" WHERE "bs"."a_id" = ? 
  LIMIT ?  [["a_id", 4], ["LIMIT", 1]]
~~~
</td>
<td>

Access with two sequential selects.

~~~ruby
B.find(3).a
  B Load (0.2ms)  SELECT "bs".* FROM "bs" WHERE "bs"."id" = ? 
  LIMIT ?  [["id", 3], ["LIMIT", 1]]
  A Load (0.1ms)  SELECT "as".* FROM "as" WHERE "as"."id" = ? 
  LIMIT ?  [["id", 4], ["LIMIT", 1]]
~~~
</td>
</tr>

<tr>
<td>what</td>
<td>

~~~ruby
~~~
</td>
<td>

~~~ruby
~~~
</td>
</tr>
</tbody>
</table>



## Instance variables and class instance variables

This is totally confusing when coming from another programming language. I found a fairly good explanation of instance variables and class instance variables [here](https://fernandobasso.dev/ruby/classes-and-objects.html).


[quickref]: https://www.zenspider.com/ruby/quickref.html#types