---
title: Rails app with Azure Entra-ID authentication
date: 2025-02-09
draft: false
categories:
- programming
tags:
- rails
- authentication
- Entra-ID
- Azure
showToc: true
---

This post demonstrates how to use Azure Entra-ID for authentication. According to [What is Microsoft Entra ID?](https://learn.microsoft.com/en-us/entra/fundamentals/whatis), 

> Microsoft Entra ID is a cloud-based identity and access management service that your employees can use to access external resources. 

Entra-ID (formerly known as Azure Active Directory) is Microsoft's brand for identity services like authentication.

In this example, we will be using Azure Entra-ID, and the [omniauth] library with the [Entra-ID][omniauth-entra-id] provider.

> OmniAuth is a library that standardizes multi-provider authentication for web applications. It was created to be powerful, flexible, and do as little as possible. Any developer can create strategies for OmniAuth that can authenticate users via disparate systems. OmniAuth strategies have been created for everything from Facebook to LDAP.

[omniauth]: https://github.com/omniauth/omniauth

[omniauth-entra-id]: https://github.com/RIPAGlobal/omniauth-entra-id

The following steps show how to use **Entra ID** and **Omniauth** for authentication with a rails application.


## Step 1: Register an Application in Microsoft Entra ID

1. **Go to the Microsoft Entra Portal**: [https://entra.microsoft.com](https://entra.microsoft.com) (this is now the entry point for Azure AD services).
   
2. **Register your app**:
   - In the **Entra ID** section, navigate to **App registrations** > **New registration**.
   - Provide a name for the app.
   - Set the **Redirect URI** to something like `http://localhost:3000/auth/entra_id/callback` if you're developing locally.
   - Add another **Redirect URI** for your production environment
   
3. After registering, note the following information:
   - **Application (client) ID**.
   - **Directory (tenant) ID**.
   
4. **Create a Client Secret**:
   - Go to **Certificates & secrets** > **New client secret**.
   - Make sure to copy the secret **value** and store it safely.
   
## Step 2: Add Required Gems to Your Rails App

In your Rails app, you'll need the `omniauth-entra_oauth2` gem.

Add the following gems to your `Gemfile`:

```bash
bundle add omniauth-entra-id
bundle add omniauth
bundle add dotenv-rails --group=development,test
```

Run `bundle install` to install the gems.

### Step 3: Configure OmniAuth with Entra ID

Create a new OmniAuth initializer for Entra ID configuration.

Create a file `config/initializers/omniauth.rb`:

```ruby
Rails.application.config.middleware.use OmniAuth::Builder do
  provider :developer, fields: [ :name, :email ]
  provider :entra_id, {
    client_id: ENV["AZURE_CLIENT_ID"],
    client_secret: ENV["AZURE_CLIENT_SECRET_VALUE"],
    tenant_id: ENV["AZURE_TENANT_ID"],
    scope: "openid email profile User.Read",
    response_type: "code",
    grant_type: "authorization_code"
  }
end

OmniAuth.config.logger = Rails.logger

# Ensure OmniAuth works in development without callback issues
OmniAuth.config.allowed_request_methods = [ :post, :get ]
OmniAuth.config.silence_get_warning = true
```

Ensure that the environment variables for Entra ID are set. Create or update a `.env` file:

```plaintext
AZURE_CLIENT_ID=your-client-id
AZURE_CLIENT_SECRET_VALUE=your-client-secret
AZURE_TENANT_ID=your-tenant-id
```

### Step 4: Set Up the Sessions Controller

Next, let's set up a `SessionsController` to handle login and logout. We’ll use OmniAuth to handle the OAuth2 authentication flow.

Run the following to generate the controller:

```bash
rails generate controller Sessions new create destroy
```

Edit the `app/controllers/sessions_controller.rb`:

```ruby
class SessionsController < ApplicationController
  def new
  end

  def create
    auth = request.env["omniauth.auth"]
    # could add guard if auth is nil

    user = User.find_or_create_by!(provider: auth["provider"], uid: auth["uid"]) do |u|
      u.first_name = auth["info"]["first_name"]
      u.last_name = auth["info"]["last_name"]
      u.email = auth["info"]["email"]
    end

    session[:user_id] = user.id
    redirect_to root_path, notice: "Signed in #{user.email} successfully"
  end

  def destroy
    session[:user_id] = nil
    redirect_to root_path, notice: "Signed out successfully"
  end
end
```

### Step 5: Add Routes for Authentication

Update your `config/routes.rb` to handle the authentication routes:

```ruby
Rails.application.routes.draw do
  root to: 'home#index'
  
  # Entra authentication routes
  get '/auth/:provider', to: 'sessions#new'
  get '/auth/:provider/callback', to: 'sessions#create'
  delete '/logout', to: 'sessions#destroy', as: 'logout'
end
```

### Step 6: Create a Simple Home Controller

For simplicity, let's show a home page where the user can log in and log out based on their session.

Run:

```bash
rails generate controller Home index
```

In the `app/controllers/home_controller.rb`:

```ruby
class HomeController < ApplicationController
  def index
  end
end
```

In `app/views/home/index.html.erb`, add the following to display login/logout links:

```erb
<% if session[:user_id] %>
  <p>Welcome, <%= User.find(session[:user_id]).name %>!</p>
  <%= link_to 'Sign Out', logout_path, method: :delete %>
<% else %>
  <%= link_to 'Sign in with Entra ID', '/auth/entra_oauth2' %>
<% end %>
```

### Step 7: Create the User Model

If you don't have a `User` model to store the authenticated users, create one. Run the following:

```bash
rails g model User email:string provider:string uid:string first_name:string last_name:string
rails db:migrate
```

In `app/models/user.rb`:

```ruby
class User < ApplicationRecord
  validates :email, presence: true, uniqueness: true
end
```

### Step 8: Run the Application

Now, you should be ready to test your Entra ID authentication. Run the Rails server:

```bash
rails server
```

Go to <http://localhost:3000>, and you should see a "Sign in with Entra ID" link. Click the link, and after authentication via Entra ID, you should be redirected back to the home page and see your name (if authenticated).

### Additional Configuration (Optional)

1. **Handle Authentication Failures**: You can define an `omniauth_failure` method in your `SessionsController` to handle errors:

```ruby
class SessionsController < ApplicationController
  def omniauth_failure
    redirect_to root_path, alert: "Authentication failed. Please try again."
  end
end
```

2. **Protect Routes**: If you want to restrict access to certain parts of your app only to authenticated users, you can use a before_action to ensure the user is logged in:

```ruby
class ApplicationController < ActionController::Base
  before_action :authenticate_user!

  private

  def authenticate_user!
    redirect_to root_path, alert: 'Please sign in first' unless session[:user_id]
  end
end
```


