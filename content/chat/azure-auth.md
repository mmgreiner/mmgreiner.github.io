Rails 8 ships with **Authentication Generator** (the new built-in auth scaffold), so the integration is a bit different. Here's the Rails 8-specific approach:

## 1. Generate the Built-in Auth (if you haven't yet)

```bash
rails generate authentication
rails db:migrate
```

This creates `User`, `Session` models and a `SessionsController` for you.

## 2. Add Gems

```ruby
# Gemfile
gem 'omniauth-azure-activedirectory-v2'
gem 'omniauth-rails_csrf_protection'
```

```bash
bundle install
```

## 3. Configure OmniAuth

```ruby
# config/initializers/omniauth.rb
OmniAuth.config.allowed_request_methods = [:post]

Rails.application.config.middleware.use OmniAuth::Builder do
  provider :azure_activedirectory_v2,
    client_id:     ENV['AZURE_CLIENT_ID'],
    client_secret: ENV['AZURE_CLIENT_SECRET'],
    tenant_id:     ENV['AZURE_TENANT_ID']
end
```

## 4. Add OmniAuth Fields to User

```bash
rails generate migration AddOmniauthToUsers provider:string uid:string name:string
rails db:migrate
```

```ruby
# app/models/user.rb
class User < ApplicationRecord
  has_many :sessions, dependent: :destroy
  normalizes :email_address, with: -> e { e.strip.downcase }

  def self.find_or_create_from_omniauth(auth)
    find_or_create_by(provider: auth.provider, uid: auth.uid) do |user|
      user.name          = auth.info.name
      user.email_address = auth.info.email
      user.password      = SecureRandom.hex(20) # required — won't be used
    end
  end
end
```

## 5. Add Routes

```ruby
# config/routes.rb
get  '/auth/:provider/callback', to: 'omniauth_callbacks#create'
get  '/auth/failure',            to: 'omniauth_callbacks#failure'
```

## 6. Create the Callbacks Controller

```ruby
# app/controllers/omniauth_callbacks_controller.rb
class OmniauthCallbacksController < ApplicationController
  allow_unauthenticated_access  # Rails 8 built-in auth method

  def create
    auth = request.env['omniauth.auth']
    @user = User.find_or_create_from_omniauth(auth)

    start_new_session_for(@user)  # Rails 8 built-in helper
    redirect_to root_path, notice: "Signed in as #{@user.name}"
  end

  def failure
    redirect_to new_session_path, alert: "Microsoft authentication failed"
  end
end
```

## 7. Update the Sessions Controller

Allow unauthenticated access and add a Microsoft login option:

```ruby
# app/controllers/sessions_controller.rb
class SessionsController < ApplicationController
  allow_unauthenticated_access only: %i[new create]
  # rest of the generated code stays the same ...
end
```

## 8. Add the Login Button to the Login View

```erb
<%# app/views/sessions/new.html.erb %>

<%# ... existing email/password form ... %>

<hr>
<%= button_to "Sign in with Microsoft", "/auth/azure_activedirectory_v2", method: :post %>
```

---

## Azure Portal Setup (reminder)

Add these redirect URIs in your app registration:
- **Dev:** `http://localhost:3000/auth/azure_activedirectory_v2/callback`
- **Prod:** `https://yourdomain.com/auth/azure_activedirectory_v2/callback`

---

## Optional: Microsoft-only (no password login)

If you want **Microsoft as the only login method**, you can remove the email/password form and make the `password` column nullable in the migration:

```bash
rails generate migration MakePasswordDigestOptional
```

```ruby
def change
  change_column_null :users, :password_digest, true
end
```

This keeps you fully on the Rails 8 auth scaffold while delegating identity to Microsoft. Let me know if you run into any issues!