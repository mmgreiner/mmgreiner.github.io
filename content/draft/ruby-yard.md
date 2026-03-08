# YARD Documentation in Ruby on Rails

YARD is the standard documentation tool for Ruby projects, including Rails apps. Here's a quick rundown.

---

## Installation

Add to your `Gemfile`:

```ruby
gem 'yard', group: :development
```

Then run `bundle install`.

---

## Basic Syntax

YARD comments use `#` with tags prefixed by `@`. Place them directly above the method or class.

```ruby
# Calculates the total price including tax.
#
# @param price [Float] the base price of the item
# @param tax_rate [Float] the tax rate as a decimal (e.g., 0.08 for 8%)
# @return [Float] the total price after tax
def total_price(price, tax_rate)
  price * (1 + tax_rate)
end
```

---

## Common Tags

| Tag | Purpose |
|---|---|
| `@param name [Type]` | Describes a method parameter |
| `@return [Type]` | Describes the return value |
| `@raise [ExceptionClass]` | Documents exceptions raised |
| `@example` | Provides a usage example |
| `@deprecated` | Marks something as deprecated |
| `@see` | Cross-references another method or URL |
| `@note` | Adds a note |
| `@author` | Credits the author |

---

## Documenting a Rails Model

```ruby
# Represents a user in the application.
#
# @attr [String] name the full name of the user
# @attr [String] email the user's unique email address
# @attr [DateTime] created_at when the record was created
class User < ApplicationRecord
  # Finds all users with admin privileges.
  #
  # @return [ActiveRecord::Relation<User>] collection of admin users
  scope :admins, -> { where(role: 'admin') }

  # Sends a welcome email to the user.
  #
  # @raise [ArgumentError] if the user has no email
  # @return [Boolean] true if the email was sent successfully
  def send_welcome_email
    raise ArgumentError, "No email address" if email.blank?
    UserMailer.welcome(self).deliver_later
    true
  end
end
```

---

## Documenting a Controller

```ruby
# Handles all API endpoints for user management.
class UsersController < ApplicationController
  # Returns a paginated list of users.
  #
  # @example GET /users?page=2
  # @return [void] renders JSON array of users
  def index
    @users = User.page(params[:page])
    render json: @users
  end
end
```

---

## Generating Documentation

```bash
# Generate docs into the /doc folder
bundle exec yard doc

# Run a local doc server at localhost:8808
bundle exec yard server

# Document a specific file
bundle exec yard doc app/models/user.rb
```

---

## `.yardopts` File

Create a `.yardopts` file in your project root to set default options:

```
app/**/*.rb
lib/**/*.rb
--output-dir docs/
--title "My Rails App"
--readme README.md
```

---

## Quick Tips

- YARD will pick up `attr_accessor`, `has_many`, etc. automatically in many cases, but explicit `@attr` tags give you more control.
- Use `@api private` to mark internal methods you don't want exposed in the public docs.
- You can use Markdown inside your YARD comments for richer formatting.

That covers the essentials to get you documenting a Rails project cleanly with YARD!