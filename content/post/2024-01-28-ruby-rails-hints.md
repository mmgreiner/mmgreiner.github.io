---
categories:
- programming
date: "2024-01-28"
draft: false
tags:
- ruby
- rails
- database
title: Ruby on Rails Tips and Tricks
showToc: true
---

This gives some tips and trick for Ruby on Rails.

# Quickref

I found an excellent quick reference guide to Ruby on [zenspider][quickref].


# ActiveRecords associations

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



# Instance variables and class instance variables

This is totally confusing when coming from another programming language. I found a fairly good explanation of instance variables and class instance variables [here](https://fernandobasso.dev/ruby/classes-and-objects.html).


[quickref]: https://www.zenspider.com/ruby/quickref.html#types