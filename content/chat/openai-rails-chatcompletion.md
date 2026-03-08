---
title: OpenAI and Rails
description: A chat on using OpenAI with chat completion and tool calling
date: 2026-02-12
draft: tfalsee
categories:
- programming
tags:
- ruby
- rails
- openai
showToc: true
---

# Small example 

Below is a **complete small Ruby on Rails example** showing how to implement **OpenAI tool calling** end-to-end.

This example:

* Uses the official OpenAI Ruby SDK
* Exposes `/v1/chat/completions`
* Lets the model call 3 tools:

  * `list_documents`
  * `get_document`
  * `add_document`
* Executes the tool in Rails
* Sends the tool result back to the model
* Returns the final assistant response

---

## 1️⃣ Gemfile

```ruby
# Gemfile
gem "openai"
```

Then:

```bash
bundle install
```

---

## 2️⃣ Document Model

```bash
rails g model Document author:string description:text checked_out:boolean
rails db:migrate
```

Model:

```ruby
# app/models/document.rb
class Document < ApplicationRecord
  validates :author, presence: true
end
```

---

## 3️⃣ Route

```ruby
# config/routes.rb
post "/v1/chat/completions", to: "chat#completions"
```

---

## 4️⃣ Controller (FULL WORKING EXAMPLE)

```ruby
# app/controllers/chat_controller.rb
class ChatController < ApplicationController
  protect_from_forgery with: :null_session

  def completions
    client = OpenAI::Client.new(api_key: ENV["OPENAI_API_KEY"])

    messages = params[:messages]

    # First call to model with tool definitions
    response = client.chat.completions(
      model: "gpt-4o-mini",
      messages: messages,
      tools: tool_definitions
    )

    message = response.dig("choices", 0, "message")

    # If model wants to call a tool
    if message["tool_calls"]
      tool_call = message["tool_calls"].first
      tool_name = tool_call["function"]["name"]
      arguments = JSON.parse(tool_call["function"]["arguments"])

      result = execute_tool(tool_name, arguments)

      # Send tool result back to model
      second_response = client.chat.completions(
        model: "gpt-4o-mini",
        messages: messages + [
          message,
          {
            role: "tool",
            tool_call_id: tool_call["id"],
            content: result.to_json
          }
        ]
      )

      render json: second_response
    else
      render json: response
    end
  end

  private

  # -------------------------
  # TOOL EXECUTION
  # -------------------------

  def execute_tool(name, args)
    case name
    when "list_documents"
      Document.all.map { |d| serialize(d) }

    when "get_document"
      doc = Document.find_by(id: args["id"])
      doc ? serialize(doc) : { error: "Not found" }

    when "add_document"
      doc = Document.create(
        author: args["author"],
        description: args["description"],
        checked_out: args["checked_out"] || false
      )

      doc.persisted? ? serialize(doc) : { error: doc.errors.full_messages }

    else
      { error: "Unknown tool" }
    end
  end

  # -------------------------
  # TOOL DEFINITIONS
  # -------------------------

  def tool_definitions
    [
      {
        type: "function",
        function: {
          name: "list_documents",
          description: "Fetch all documents",
          parameters: {
            type: "object",
            properties: {}
          }
        }
      },
      {
        type: "function",
        function: {
          name: "get_document",
          description: "Fetch a document by ID",
          parameters: {
            type: "object",
            properties: {
              id: { type: "integer" }
            },
            required: ["id"]
          }
        }
      },
      {
        type: "function",
        function: {
          name: "add_document",
          description: "Add a new document",
          parameters: {
            type: "object",
            properties: {
              author: { type: "string" },
              description: { type: "string" },
              checked_out: { type: "boolean" }
            },
            required: ["author"]
          }
        }
      }
    ]
  end

  def serialize(doc)
    {
      id: doc.id,
      author: doc.author,
      description: doc.description,
      checked_out: doc.checked_out
    }
  end
end
```

---

## 5️⃣ Example Request (curl)

```bash
curl -X POST http://localhost:3000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      { "role": "user", "content": "Add a document by Alice saying Hello World" }
    ]
  }' | jq
```

The model will:

1. Detect it needs `add_document`
2. Call the tool
3. Rails executes it
4. Rails sends result back to model
5. Model generates final answer

---

## 6️⃣ How This Works Internally

Step-by-step:

1. User sends prompt
2. Rails sends prompt + tools to OpenAI
3. Model returns:

```json
{
  "tool_calls": [
    {
      "function": {
        "name": "add_document",
        "arguments": "{ \"author\": \"Alice\", \"description\": \"Hello World\" }"
      }
    }
  ]
}
```

4. Rails runs `add_document`
5. Rails sends result back to model
6. Model returns final human-readable response

---

## 7️⃣ Why This Works With Open WebUI / Jan

Because this endpoint behaves like:

```
POST /v1/chat/completions
```

Which is what:

* Open WebUI
* Jan
* Most OpenAI-compatible clients

expect.

No MCP needed.

---

## 🏁 Final Architecture

```
Open WebUI
    ↓
Rails /v1/chat/completions
    ↓
OpenAI model
    ↓
Rails tool execution
    ↓
Final response
```

---

If you'd like next, I can:

* Add streaming support
* Add authentication
* Add file uploads (ActiveStorage)
* Add pagination and search
* Refactor into service objects (production-ready structure)
* Show how to run this fully offline with Ollama

Just tell me your target setup.
Perfect — now we’ll make this **production-ready**:

We will:

1. ✅ Add API key authentication
2. ✅ Move OpenAI logic into a service object
3. ✅ Move tool execution into a dedicated service
4. ✅ Keep controller thin (Rails best practice)
5. ✅ Keep OpenAI-compatible `/v1/chat/completions` endpoint

---

## ✅ Final Architecture

```
POST /v1/chat/completions
        ↓
ChatController
        ↓
OpenAIChatService
        ↓
ToolExecutorService
        ↓
Document model
```

Clean separation:

* Controller = HTTP layer
* OpenAIChatService = LLM orchestration
* ToolExecutorService = business logic
* Model = persistence

---

## 1️⃣ API Authentication (Simple Production-Safe Version)

We’ll use **Bearer token authentication**.

Add to `.env`:

```
APP_API_KEY=supersecretkey
OPENAI_API_KEY=sk-xxxx
```

---

### ApplicationController

```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::API
  before_action :authenticate!

  private

  def authenticate!
    header = request.headers["Authorization"]
    token = header&.split(" ")&.last

    unless ActiveSupport::SecurityUtils.secure_compare(
      token.to_s,
      ENV["APP_API_KEY"].to_s
    )
      render json: { error: "Unauthorized" }, status: :unauthorized
    end
  end
end
```

Now all endpoints require:

```
Authorization: Bearer supersecretkey
```

---

## 2️⃣ Thin Controller

```ruby
# app/controllers/chat_controller.rb
class ChatController < ApplicationController
  def completions
    result = OpenAIChatService.new(params[:messages]).call
    render json: result
  end
end
```

That’s it.

---

## 3️⃣ OpenAIChatService

```ruby
# app/services/openai_chat_service.rb
class OpenAIChatService
  def initialize(messages)
    @messages = messages
    @client = OpenAI::Client.new(api_key: ENV["OPENAI_API_KEY"])
  end

  def call
    first_response = @client.chat.completions(
      model: "gpt-4o-mini",
      messages: @messages,
      tools: ToolExecutorService.tool_definitions
    )

    message = first_response.dig("choices", 0, "message")

    return first_response unless message["tool_calls"]

    handle_tool_call(message)
  end

  private

  def handle_tool_call(message)
    tool_call = message["tool_calls"].first
    tool_name = tool_call.dig("function", "name")
    arguments = JSON.parse(tool_call.dig("function", "arguments"))

    tool_result = ToolExecutorService.new.execute(tool_name, arguments)

    @client.chat.completions(
      model: "gpt-4o-mini",
      messages: @messages + [
        message,
        {
          role: "tool",
          tool_call_id: tool_call["id"],
          content: tool_result.to_json
        }
      ]
    )
  end
end
```

---

## 4️⃣ ToolExecutorService

```ruby
# app/services/tool_executor_service.rb
class ToolExecutorService
  def self.tool_definitions
    [
      {
        type: "function",
        function: {
          name: "list_documents",
          description: "Fetch all documents",
          parameters: {
            type: "object",
            properties: {}
          }
        }
      },
      {
        type: "function",
        function: {
          name: "get_document",
          description: "Fetch a document by ID",
          parameters: {
            type: "object",
            properties: {
              id: { type: "integer" }
            },
            required: ["id"]
          }
        }
      },
      {
        type: "function",
        function: {
          name: "add_document",
          description: "Add a new document",
          parameters: {
            type: "object",
            properties: {
              author: { type: "string" },
              description: { type: "string" },
              checked_out: { type: "boolean" }
            },
            required: ["author"]
          }
        }
      }
    ]
  end

  def execute(name, args)
    case name
    when "list_documents"
      Document.all.map { |d| serialize(d) }

    when "get_document"
      doc = Document.find_by(id: args["id"])
      doc ? serialize(doc) : { error: "Not found" }

    when "add_document"
      doc = Document.create(
        author: args["author"],
        description: args["description"],
        checked_out: args["checked_out"] || false
      )

      doc.persisted? ? serialize(doc) : { error: doc.errors.full_messages }

    else
      { error: "Unknown tool" }
    end
  end

  private

  def serialize(doc)
    {
      id: doc.id,
      author: doc.author,
      description: doc.description,
      checked_out: doc.checked_out
    }
  end
end
```

---

## 5️⃣ Routes

```ruby
# config/routes.rb
post "/v1/chat/completions", to: "chat#completions"
```

---

## 6️⃣ Example Production Curl

```bash
curl -X POST http://localhost:3000/v1/chat/completions \
  -H "Authorization: Bearer supersecretkey" \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      { "role": "user", "content": "Add a document by Alice saying Hello World" }
    ]
  }' | jq
```

---

## ✅ What We Improved

| Before            | Now                                |
| ----------------- | ---------------------------------- |
| Fat controller    | Thin controller                    |
| No auth           | Secure bearer token                |
| Tool logic inline | Dedicated ToolExecutorService      |
| Hard to scale     | Easy to extend                     |
| Hard to test      | Each service testable in isolation |

---

## 🧠 Why This Is Production-Ready

* Separation of concerns
* Secure token validation
* Single responsibility services
* Compatible with:

  * Open WebUI
  * Jan
  * Custom frontends
  * CLI clients
* Easily swappable model (OpenAI → Ollama)

---

## 🚀 If You Want To Go Even Further

Next production upgrades could be:

* ✅ Streaming responses
* ✅ Logging tool calls
* ✅ Rate limiting
* ✅ Multi-user authentication (JWT)
* ✅ Background job execution
* ✅ Multi-tool chaining
* ✅ File uploads via ActiveStorage
* ✅ Multi-tenant document scoping
* ✅ Replace OpenAI with local LLM
* ✅ Convert to full Agent architecture

---

If you'd like, I can now give you:

* A **fully Open-WebUI compatible response schema**
* Or the **same architecture implemented using MCP instead of OpenAI tools**
* Or how to make this work with Ollama locally**
* Or how to add document file attachments properly**

What direction are you building toward?

# Using authentication 

Perfect — since you're using **Rails 8 authentication (the built-in generator, not Devise)**, the clean way to integrate it is:

👉 Keep session-based auth for browser users
👉 Add token-based auth for API clients (Open WebUI / curl / agents)
👉 Scope documents to `current_user`

Below is a **production-clean Rails 8 setup** that works for:

* Browser login
* API calls
* Open WebUI
* Ollama
* Tool calling
* Multi-user isolation

---

## 🧱 Assumption

You generated auth using:

```bash
rails generate authentication
```

So you now have:

* `User`
* `Session`
* `Current`
* `Authentication` concern
* `current_user`

---

## 🎯 Goal

Make `/v1/chat/completions`:

* Require authentication
* Work with:

  * Browser session
  * API Bearer token
* Scope documents to `current_user`

---

## ✅ Step 1 — Add API Token to User

Migration:

```bash
rails g migration AddApiTokenToUsers api_token:string:index
rails db:migrate
```

---

### app/models/user.rb

```ruby
class User < ApplicationRecord
  has_secure_password
  has_many :documents

  before_create :generate_api_token

  private

  def generate_api_token
    self.api_token ||= SecureRandom.hex(32)
  end
end
```

Now every user gets a permanent API token.

---

## ✅ Step 2 — Support Token OR Session Authentication

Modify `ApplicationController`.

---

### app/controllers/application_controller.rb

```ruby
class ApplicationController < ActionController::Base
  include Authentication

  before_action :authenticate_with_api_token

  private

  def authenticate_with_api_token
    return if current_user.present?

    auth_header = request.headers["Authorization"]
    return unless auth_header&.start_with?("Bearer ")

    token = auth_header.split(" ").last
    user = User.find_by(api_token: token)

    if user
      Current.user = user
    end
  end
end
```

What this does:

* If logged in via session → works
* If request has `Authorization: Bearer ...` → sets `Current.user`
* Otherwise → unauthenticated

---

## ✅ Step 3 — Require Auth on API

In `ChatController`:

```ruby
class ChatController < ApplicationController
  before_action :require_authentication

  def completions
    result = OpenAIChatService.new(params[:messages], current_user).call
    render json: result
  end

  private

  def require_authentication
    unless current_user
      render json: { error: "Unauthorized" }, status: :unauthorized
    end
  end
end
```

---

## ✅ Step 4 — Scope Documents to Current User

Update `Document`:

```ruby
class Document < ApplicationRecord
  belongs_to :user
end
```

Migration if needed:

```bash
rails g migration AddUserToDocuments user:references
rails db:migrate
```

---

## ✅ Step 5 — Pass User into Service

Update service:

```ruby
class OpenAIChatService
  def initialize(messages, user)
    @messages = messages
    @user = user
    @client = OpenAI::Client.new(
      api_key: "ollama",
      uri_base: ENV.fetch("OPENAI_BASE_URL", "http://localhost:11434/v1")
    )
  end
```

---

## ✅ Step 6 — Scope Tool Execution

Update `ToolExecutorService`:

```ruby
class ToolExecutorService
  def initialize(user)
    @user = user
  end

  def execute(name, args)
    case name
    when "list_documents"
      @user.documents.map { |d| serialize(d) }

    when "get_document"
      doc = @user.documents.find_by(id: args["id"])
      doc ? serialize(doc) : { error: "Not found" }

    when "add_document"
      doc = @user.documents.create(
        author: args["author"],
        description: args["description"],
        checked_out: args["checked_out"] || false
      )

      doc.persisted? ? serialize(doc) : { error: doc.errors.full_messages }
    else
      { error: "Unknown tool" }
    end
  end
```

And update the call:

```ruby
tool_result = ToolExecutorService.new(@user).execute(tool_name, arguments)
```

---

## 🧠 Now You Have

| Feature              | Works |
| -------------------- | ----- |
| Browser login        | ✅     |
| API token auth       | ✅     |
| Open WebUI           | ✅     |
| curl                 | ✅     |
| Multi-user isolation | ✅     |
| Ollama local         | ✅     |
| Cloud OpenAI         | ✅     |

---

## 🔑 How API Users Authenticate

User checks their token:

```ruby
current_user.api_token
```

Then:

```bash
curl -X POST http://localhost:3000/v1/chat/completions \
  -H "Authorization: Bearer USER_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      { "role": "user", "content": "List my documents" }
    ]
  }'
```

---

## 🏗 Clean Production Architecture

```
User (session or token)
        ↓
Rails Authentication
        ↓
ChatController
        ↓
OpenAIChatService
        ↓
ToolExecutorService (scoped to user)
        ↓
User's Documents
```

---

## 🚀 Optional Production Upgrades

You may also want:

* Expiring API tokens
* Multiple API tokens per user
* Admin-only tools
* Rate limiting per user
* Audit log of tool calls
* Organization/multi-tenant scoping

---

If you want next, I can show:

* 🔐 How to create multiple API keys per user (like OpenAI)
* 🏢 How to add organization/team isolation
* 📊 How to log every tool call in a database
* ⚡ How to add streaming responses
* 🧠 How to convert this into an MCP server instead

What level are you building this for — internal tool, SaaS, or enterprise?
