---
title: The grape gem for Rest APIs
description: A short example of using the grape gem to define REST APIs in Ruby
date: 2026-04-16
draft: true
categories:
- programming
tags:
- ruby
- REST API
- OpenAI
- LLM
showToc: true
---

## Purpose

A summary of Claude ✨ chats on how to build a REST API with the following characteristics:

- integrated into Rails
- generates Swagger documentation to be used with postman
- also possible to use with OpenAI LLM summaries.


## REST API with Grape in Rails 8

Here's the clearest path for your setup:

### Do you need those gems?

- **`grape`** — yes, required
- **`grape-entity`** — optional, but **recommended**. It controls what fields are exposed in the API response (like a serializer/presenter layer). Without it, you'd manually `present` raw ActiveRecord objects.
- **`grape-swagger`** — yes, if you want Swagger/OpenAPI docs
- **`grape-swagger-entity`** — needed to bridge `grape-entity` and `grape-swagger` so Swagger can infer response schemas from your entities

### Gemfile

```ruby
gem "grape"
gem "grape-entity"
gem "grape-swagger"
gem "grape-swagger-entity"
```

---

### 1. The Entity (`app/api/entities/post_entity.rb`)

```ruby
module Entities
  class Post < Grape::Entity
    expose :id
    expose :title
    expose :body
    expose :created_at
  end
end
```

---

### 2. The API (`app/api/api/v1/posts.rb`)

```ruby
module API
  module V1
    class Posts < Grape::API
      resource :posts do

        desc "List all posts" do
          success Entities::Post
          is_array true
        end
        get do
          present Post.all, with: Entities::Post
        end

        desc "Get a post" do
          success Entities::Post
        end
        params do
          requires :id, type: Integer, desc: "Post ID"
        end
        get ":id" do
          present Post.find(params[:id]), with: Entities::Post
        end

        desc "Create a post" do
          success Entities::Post
        end
        params do
          requires :title, type: String, desc: "Post title"
          requires :body,  type: String, desc: "Post body"
        end
        post do
          present Post.create!(declared(params)), with: Entities::Post
        end

        desc "Update a post" do
          success Entities::Post
        end
        params do
          requires :id,    type: Integer, desc: "Post ID"
          optional :title, type: String,  desc: "Post title"
          optional :body,  type: String,  desc: "Post body"
        end
        put ":id" do
          post = Post.find(params[:id])
          post.update!(declared(params, include_missing: false))
          present post, with: Entities::Post
        end

        desc "Delete a post"
        params do
          requires :id, type: Integer, desc: "Post ID"
        end
        delete ":id" do
          Post.find(params[:id]).destroy!
          { message: "Post deleted" }
        end

      end
    end
  end
end
```

---

### 3. The Base API with Swagger (`app/api/api/base.rb`)

```ruby
module API
  class Base < Grape::API
    format :json
    prefix :api

    mount API::V1::Posts

    # Mount Swagger docs last
    add_swagger_documentation(
      api_version: "v1",
      hide_documentation_path: true,
      mount_path: "/swagger_doc",
      info: {
        title: "My Rails API",
        description: "Post CRUD API"
      }
    )
  end
end
```

---

### 4. Mount in Rails (`config/routes.rb`)

```ruby
Rails.application.routes.draw do
  mount API::Base => "/"
end
```

---

### 5. Autoload the `api/` folder (`config/application.rb`)

```ruby
config.paths.add "app/api", glob: "**/*.rb"
config.autoload_paths += Dir["#{root}/app/api/*"]
```

---

### Accessing Swagger in Postman

1. Start your server: `rails s`
2. The Swagger JSON will be at: **`http://localhost:3000/api/swagger_doc`**
3. In Postman: **Import → Link** → paste that URL → Postman generates a full collection

---

### Summary

| Gem | Needed? | Why |
|---|---|---|
| `grape` | ✅ Yes | Core API framework |
| `grape-entity` | ✅ Recommended | Clean response shaping |
| `grape-swagger` | ✅ Yes | Generates Swagger/OpenAPI JSON |
| `grape-swagger-entity` | ✅ Yes | Links entities to Swagger schemas |


This is the **standard minimal stack** — no extra tooling needed. `grape-swagger-rails` is an optional 5th gem if you also want a Swagger UI served directly from Rails, but for Postman you don't need it.

## Grape and ActiveRecords

> Do I need to define the Grape Entities or can I take it from the ActiveRecord definition?

Yes, you do — but that's **intentional and actually desirable**. They serve completely different purposes:

### Two different concerns

| | ActiveRecord Model | Grape Entity |
|---|---|---|
| **Purpose** | Database mapping, business logic, validations | API response shaping / serialization |
| **Audience** | Your app internals | External API consumers |
| **Controls** | What's *stored* | What's *exposed* |

---

### The key benefit: decoupling

Your `Post` model might have sensitive or internal fields you never want to expose:

```ruby
# ActiveRecord — everything in the DB
class Post < ApplicationRecord
  # has: id, title, body, created_at, updated_at,
  #      author_id, internal_notes, admin_flag, stripe_token ...
end
```

```ruby
# Grape Entity — only what the API consumer should see
class Entities::Post < Grape::Entity
  expose :id
  expose :title
  expose :body
  expose :created_at
  # stripe_token, admin_flag, internal_notes → never exposed
end
```

Without the entity layer, you'd have to remember to manually blacklist sensitive fields every time — which is error-prone.

---

### It's the same pattern everywhere

This "declare twice" pattern is not unique to Grape — it's a universal API design principle:

- **ActiveModelSerializers** → separate serializer class
- **Jbuilder** → separate `.json.jbuilder` view
- **Blueprinter / Panko** → separate blueprint class
- **GraphQL-Ruby** → separate type class

Grape Entity is just Grape's version of a **serializer/presenter layer**. The duplication is the point — your DB schema and your API contract should be able to **evolve independently**.

---

### When it feels like too much duplication

If your API truly mirrors your model 1:1 and you have no sensitive fields, you can skip `grape-entity` entirely and just render directly:

```ruby
get do
  Post.all.as_json(only: [:id, :title, :body, :created_at])
end
```

But as soon as your app grows (nested associations, computed fields, versioned APIs), you'll want the entity layer back.## Testing Grape APIs with Minitest in Rails 8

## Testing Grape APIs

Grape APIs are Rack apps, so you test them by making HTTP requests directly against the mounted API — no need for extra gems.

---

### Setup: Base API Test Helper (`test/test_helper.rb`)

Add Rack::Test to your existing test helper:

```ruby
ENV["RAILS_ENV"] ||= "test"
require_from_root "config/environment"
require "rails/test_help"
require "rack/test"  # comes with Rails, no extra gem needed

class ActiveSupport::TestCase
  fixtures :all
end

# Separate base class for API tests
class ApiTestCase < ActiveSupport::TestCase
  include Rack::Test::Methods

  def app
    API::Base  # your Grape base class
  end

  # Helper to parse JSON responses
  def json_response
    JSON.parse(last_response.body)
  end
end
```

---

### The Tests (`test/api/v1/posts_test.rb`)

```ruby
require "test_helper"

class API::V1::PostsTest < ApiTestCase

  setup do
    @post = Post.create!(title: "Hello", body: "World")
  end

  teardown do
    Post.delete_all
  end

  # --- GET /api/posts ---

  test "GET /api/posts returns all posts" do
    get "/api/posts"

    assert_equal 200, last_response.status
    assert_equal 1, json_response.length
    assert_equal "Hello", json_response.first["title"]
  end

  # --- GET /api/posts/:id ---

  test "GET /api/posts/:id returns a post" do
    get "/api/posts/#{@post.id}"

    assert_equal 200, last_response.status
    assert_equal @post.id,    json_response["id"]
    assert_equal "Hello",     json_response["title"]
    assert_equal "World",     json_response["body"]
  end

  test "GET /api/posts/:id returns 404 for missing post" do
    get "/api/posts/99999"

    assert_equal 404, last_response.status
  end

  # --- POST /api/posts ---

  test "POST /api/posts creates a post" do
    assert_difference "Post.count", 1 do
      post "/api/posts", { title: "New", body: "Content" }
    end

    assert_equal 201, last_response.status
    assert_equal "New", json_response["title"]
  end

  test "POST /api/posts returns 400 when params are missing" do
    post "/api/posts", { title: "No body here" }

    assert_equal 400, last_response.status
    assert_match "body", json_response["error"]  # Grape param error message
  end

  # --- PUT /api/posts/:id ---

  test "PUT /api/posts/:id updates a post" do
    put "/api/posts/#{@post.id}", { title: "Updated" }

    assert_equal 200, last_response.status
    assert_equal "Updated", json_response["title"]
    assert_equal "World",   json_response["body"]   # unchanged
  end

  test "PUT /api/posts/:id returns 404 for missing post" do
    put "/api/posts/99999", { title: "Ghost" }

    assert_equal 404, last_response.status
  end

  # --- DELETE /api/posts/:id ---

  test "DELETE /api/posts/:id deletes a post" do
    assert_difference "Post.count", -1 do
      delete "/api/posts/#{@post.id}"
    end

    assert_equal 200, last_response.status
    assert_equal "Post deleted", json_response["message"]
  end

  test "DELETE /api/posts/:id returns 404 for missing post" do
    delete "/api/posts/99999"

    assert_equal 404, last_response.status
  end

end
```

---

### Testing with JSON request body (for POST/PUT)

Rack::Test sends form-encoded params by default. If your API expects `Content-Type: application/json`:

```ruby
# Add this helper to ApiTestCase
def post_json(path, params = {})
  post path, params.to_json, "CONTENT_TYPE" => "application/json"
end

def put_json(path, params = {})
  put path, params.to_json, "CONTENT_TYPE" => "application/json"
end
```

Then use them in tests:

```ruby
test "POST /api/posts creates a post with JSON body" do
  post_json "/api/posts", { title: "New", body: "Content" }

  assert_equal 201, last_response.status
end
```

---

### Key points

- **`last_response`** — the Rack response object (`.status`, `.body`, `.headers`)
- **`last_request`** — the Rack request object, useful for debugging
- **No `rails test` magic needed** — just plain Rack, so it's fast
- **Fixtures work fine** — `ApiTestCase` inherits from `ActiveSupport::TestCase` so `fixtures :all` applies

The tests run with the normal `rails test` or `bin/rails test test/api/` command.Short answer: **No, not directly** — they are completely different things serving different purposes.

---

## Grape Entities and OpenAI schemas

> Is there a duplication between the Grape API Entities and the OpenAI schemas?

### What each "entity" is for

| | Grape Entity | RubyLLM / OpenAI Schema |
|---|---|---|
| **Purpose** | Shape JSON *responses* from your API | Define structured *input/output* for LLM tool calls |
| **Consumer** | Postman, frontend, API clients | The LLM model itself |
| **Format** | Serialized Ruby object → JSON | JSON Schema (type, properties, required, description) |
| **Direction** | Outbound (your app → client) | Bidirectional (you → LLM → you) |

---

### What RubyLLM structured output looks like

With RubyLLM you define a schema the LLM must conform to:

```ruby
# This is a JSON Schema definition, not a serializer
response = RubyLLM.chat.ask(
  "Summarize this post",
  schema: {
    type: "object",
    properties: {
      summary:  { type: "string", description: "A short summary" },
      keywords: { type: "array",  items: { type: "string" } },
      sentiment: { type: "string", enum: ["positive", "negative", "neutral"] }
    },
    required: ["summary", "keywords", "sentiment"]
  }
)
```

A Grape Entity has no concept of `description`, `enum`, `required`, or JSON Schema types — it only knows how to *serialize* Ruby objects.

---

### The clean solution: a shared plain Ruby class

Define your data shape **once** as a plain Ruby object, then **derive both** from it:

```ruby
# app/models/post_summary.rb
# Plain Ruby — no framework dependency
class PostSummary
  FIELDS = {
    summary:   { type: "string",  description: "A short summary of the post" },
    keywords:  { type: "array",   description: "Key topics", items: { type: "string" } },
    sentiment: { type: "string",  description: "Tone", enum: ["positive", "negative", "neutral"] }
  }.freeze

  attr_accessor :summary, :keywords, :sentiment

  def initialize(attrs = {})
    attrs.each { |k, v| public_send(:"#{k}=", v) }
  end

  # For RubyLLM / OpenAI structured output
  def self.to_llm_schema
    {
      type: "object",
      properties: FIELDS,
      required: FIELDS.keys.map(&:to_s)
    }
  end
end
```

Then derive both from it:

```ruby
# app/api/entities/post_summary_entity.rb
module Entities
  class PostSummary < Grape::Entity
    # Drive exposed fields from the same FIELDS constant
    ::PostSummary::FIELDS.each_key do |field|
      expose field
    end
  end
end
```

```ruby
# Somewhere in a service object
class PostSummaryService
  def summarize(post)
    response = RubyLLM.chat.ask(
      "Summarize this post: #{post.body}",
      schema: PostSummary.to_llm_schema
    )

    PostSummary.new(JSON.parse(response.content))
  end
end
```

```ruby
# In your Grape API endpoint
desc "Get AI summary of a post" do
  success Entities::PostSummary
end
get ":id/summary" do
  post = Post.find(params[:id])
  summary = PostSummaryService.new.summarize(post)
  present summary, with: Entities::PostSummary
end
```

---

### The mental model

```
PostSummary::FIELDS  ← single source of truth
        │
        ├──→  Entities::PostSummary   (Grape)     → shapes your API response
        │
        └──→  PostSummary.to_llm_schema (RubyLLM) → constrains the LLM output
```

This way you define the shape **once** and both layers stay in sync automatically when you add or remove fields.
You're right, polling is not ideal. Here are the main alternatives, ordered by complexity:

---

### Option 1: Action Cable (WebSockets) — best for Rails-native
The job pushes the result to the client when done. No polling needed.

```ruby
# app/jobs/summarize_post_job.rb
def perform(post_summary_id)
  post_summary = PostSummary.find(post_summary_id)
  post_summary.update!(status: "processing")

  # ... LLM call ...

  post_summary.update!(status: "done", summary: result["summary"], ...)

  # Push to the client when done
  ActionCable.server.broadcast(
    "post_summary_#{post_summary.post_id}",
    { status: "done", summary: post_summary.summary, ... }
  )
rescue => e
  post_summary.update!(status: "failed", error_message: e.message)
  ActionCable.server.broadcast(
    "post_summary_#{post_summary.post_id}",
    { status: "failed", error: e.message }
  )
end
```

```ruby
# app/channels/post_summary_channel.rb
class PostSummaryChannel < ApplicationCable::Channel
  def subscribed
    stream_from "post_summary_#{params[:post_id]}"
  end
end
```

Client subscribes once and just waits:
```javascript
const cable = createConsumer()
cable.subscriptions.create(
  { channel: "PostSummaryChannel", post_id: 1 },
  { received: (data) => console.log("Got summary:", data) }
)
```

---

### Option 2: Webhook — best for server-to-server / Postman use cases

The client registers a callback URL upfront. Your job POSTs the result there when done.

```ruby
# Migration: add webhook_url to post_summaries
add_column :post_summaries, :webhook_url, :string
```

```ruby
# app/jobs/summarize_post_job.rb
def perform(post_summary_id)
  # ... LLM call ...

  post_summary.update!(status: "done", ...)

  # Fire webhook if registered
  if post_summary.webhook_url.present?
    WebhookDeliveryJob.perform_later(post_summary.id)
  end
end
```

```ruby
# app/jobs/webhook_delivery_job.rb
class WebhookDeliveryJob < ApplicationJob
  def perform(post_summary_id)
    post_summary = PostSummary.find(post_summary_id)

    Net::HTTP.post(
      URI(post_summary.webhook_url),
      Entities::PostSummary.represent(post_summary).to_json,
      "Content-Type" => "application/json"
    )
  end
end
```

Client registers the webhook at request time:
```bash
POST /api/posts/1/summary
{ "webhook_url": "https://myclient.com/callbacks/summary" }
```

---

### Option 3: SSE (Server-Sent Events) — best for simple browser streaming

Simpler than WebSockets — HTTP-based, one direction (server → client), no extra infrastructure.

```ruby
# app/api/api/v1/post_summaries.rb
get ":id/summary/stream" do
  post    = Post.find(params[:id])
  summary = PostSummary.find_by!(post: post)

  stream_headers = {
    "Content-Type"  => "text/event-stream",
    "Cache-Control" => "no-cache",
    "X-Accel-Buffering" => "no"  # important for Nginx
  }

  [200, stream_headers, Enumerator.new do |yielder|
    loop do
      current = PostSummary.find(summary.id)  # re-fetch each iteration

      if current.status == "done"
        yielder << "data: #{Entities::PostSummary.represent(current).to_json}\n\n"
        break
      elsif current.status == "failed"
        yielder << "data: #{{ error: current.error_message }.to_json}\n\n"
        break
      else
        yielder << "data: #{{ status: current.status }.to_json}\n\n"
        sleep 1
      end
    end
  end]
end
```

---

### Which to pick?

| | Action Cable | Webhook | SSE |
|---|---|---|---|
| **Best for** | Browser clients | Server-to-server | Simple browser streaming |
| **Infrastructure** | Redis (for multi-process) | Nothing extra | Nothing extra |
| **Postman friendly** | ❌ | ✅ | ✅ |
| **Complexity** | Medium | Low | Low |
| **Bidirectional** | ✅ | ❌ | ❌ |

For your use case — a Rails API tested in Postman but also consumed by a browser — the pragmatic choice is:

- **Webhooks** for Postman / server clients
- **Action Cable** for browser clients

Both can coexist: fire the webhook if `webhook_url` is present, broadcast via Action Cable regardless.

## Authentication

> How can I protect the Rest API?

There are several common approaches for protecting a Grape API:

**1. API Key Authentication (simplest)**
```ruby
before do
  api_key = headers['X-Api-Key'] || params[:api_key]
  error!('Unauthorized', 401) unless api_key == ENV['API_KEY']
end
```

**2. HTTP Basic Auth**
```ruby
http_basic_auth do |username, password|
  username == ENV['API_USER'] && password == ENV['API_PASSWORD']
end
```

**3. JWT Token Auth (most common for modern APIs)**

Add `gem 'jwt'` to your Gemfile, then:

```ruby
helpers do
  def authenticate!
    token = headers['Authorization']&.split(' ')&.last
    error!('Unauthorized', 401) unless token
    
    begin
      JWT.decode(token, ENV['JWT_SECRET'], true, algorithm: 'HS256')
    rescue JWT::DecodeError
      error!('Unauthorized', 401)
    end
  end
end

before do
  authenticate!
end
```

**4. Devise + Token (if using Rails + Devise)**
```ruby
before do
  token = headers['Authorization']&.split(' ')&.last
  @current_user = User.find_by(authentication_token: token)
  error!('Unauthorized', 401) unless @current_user
end
```

**Scoping auth to specific endpoints** — if you don't want to protect everything globally, use `before` inside a specific resource block rather than at the top level:

```ruby
resource :protected do
  before { authenticate! }
  
  get :secret do
    { data: 'secret stuff' }
  end
end

resource :public do
  get :info do
    { data: 'public stuff' }
  end
end
```

For most APIs, **JWT** is the recommended approach as it's stateless, widely supported, and works well across services. Use **API keys** if you're building a simpler service-to-service integration.

## Complete Flow with token generation

> How do I get the token?

For JWT, the token is generated by your auth endpoint (e.g. a login route). Here's the full flow:

**1. Add the login endpoint (generates the token)**
```ruby
resource :auth do
  desc 'Login and get a JWT token'
  params do
    requires :email, type: String
    requires :password, type: String
  end
  
  post :login do
    user = User.find_by(email: params[:email])
    
    if user&.valid_password?(params[:password])
      payload = {
        user_id: user.id,
        email: user.email,
        exp: Time.now.to_i + (24 * 3600)  # expires in 24 hours
      }
      token = JWT.encode(payload, ENV['JWT_SECRET'], 'HS256')
      { token: token }
    else
      error!('Invalid credentials', 401)
    end
  end
end
```

**2. Client calls login to get the token**
```bash
curl -X POST http://localhost:3000/api/auth/login \
  -d "email=user@example.com&password=secret"

# returns:
# { "token": "eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjox..." }
```

**3. Client sends token in subsequent requests**
```bash
curl http://localhost:3000/api/protected/secret \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9..."
```

**The JWT secret** in `ENV['JWT_SECRET']` should be a long random string — generate one with:
```bash
ruby -rsecurerandom -e 'puts SecureRandom.hex(64)'
```

And store it in your `.env` file (never commit it to git):
```
JWT_SECRET=abc123yourlongrandomsecrethere
```

The token itself contains the encoded payload (user id, expiry etc.) and is cryptographically signed with your secret — so you can verify it without a database lookup.

## Combine with Rails 8 authentication

> Can I combine this with rails 8 authentication?

Yes! Rails 8's built-in authentication generator uses session-based auth, but you can extend it to also issue JWTs for your Grape API. The key is reusing the same `User` model and password verification.

**1. Add a JWT helper module**
```ruby
# app/lib/jwt_helper.rb
module JwtHelper
  SECRET = ENV['JWT_SECRET'] || Rails.application.secret_key_base
  ALGORITHM = 'HS256'

  def self.encode(payload)
    payload[:exp] ||= 24.hours.from_now.to_i
    JWT.encode(payload, SECRET, ALGORITHM)
  end

  def self.decode(token)
    JWT.decode(token, SECRET, true, algorithm: ALGORITHM, verify_expiration: true).first
  rescue JWT::ExpiredSignature
    nil  # token has expired
  rescue JWT::DecodeError
    nil  # token is invalid
  end
end
```

**2. Add an API login endpoint in Grape**
```ruby
resource :auth do
  desc 'Login and return a JWT token'
  params do
    requires :email, type: String
    requires :password, type: String
  end

  post :login do
    # Reuses Rails 8's User model and authenticate method
    user = User.authenticate_by(
      email: params[:email],
      password: params[:password]
    )

    error!('Invalid credentials', 401) unless user

    token = JwtHelper.encode({ user_id: user.id, email: user.email })
    { token: token }
  end
end
```

**3. Protect Grape endpoints with the same User model**
```ruby
helpers do
  def authenticate!
    token = headers['Authorization']&.split(' ')&.last
    error!('Unauthorized', 401) unless token

    payload = JwtHelper.decode(token)
    error!('Unauthorized', 401) unless payload

    @current_user = User.find_by(id: payload['user_id'])
    error!('Unauthorized', 401) unless @current_user
  end

  def current_user
    @current_user
  end
end
```

**The flow is then:**

- Browser/web app → uses Rails 8 session auth as normal (cookies, `Current.user`, etc.)
- API clients → `POST /api/auth/login` to get a JWT, then use `Authorization: Bearer <token>` header

`User.authenticate_by` is the method Rails 8 generates — it handles secure password comparison via `has_secure_password`, so you're not duplicating any auth logic, just adding a JWT issuance layer on top.

## Configuration expiration date

Yes! A clean place to put it is in the `JwtHelper` module as a constant, or in `Rails.application.config` if you want it configurable per environment:

**Option 1: Constant in JwtHelper (simple)**
```ruby
module JwtHelper
  SECRET = ENV['JWT_SECRET'] || Rails.application.secret_key_base
  ALGORITHM = 'HS256'
  EXPIRATION = 24.hours

  def self.encode(payload)
    payload[:exp] ||= EXPIRATION.from_now.to_i
    JWT.encode(payload, SECRET, ALGORITHM)
  end
end
```

**Option 2: Rails config (per environment)**
```ruby
# config/application.rb
config.jwt_expiration = 24.hours

# config/environments/development.rb
config.jwt_expiration = 1.year  # less strict in dev

# config/environments/production.rb
config.jwt_expiration = 1.hour  # stricter in prod
```

Then reference it in JwtHelper:
```ruby
def self.encode(payload)
  payload[:exp] ||= Rails.application.config.jwt_expiration.from_now.to_i
  JWT.encode(payload, SECRET, ALGORITHM)
end
```

**Option 3: Environment variable (12-factor style)**
```ruby
EXPIRATION = ENV.fetch('JWT_EXPIRATION_HOURS', 24).to_i.hours
```

And in your `.env`:
```
JWT_EXPIRATION_HOURS=24
```

Option 2 is the most Rails-idiomatic since it fits naturally with environment-specific config, while Option 3 is best if you want to change expiration without a redeployment.

