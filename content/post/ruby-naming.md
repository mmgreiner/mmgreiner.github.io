---
title: Ruby gem naming convention
description: Naming convention for ruby gems which wrap Python packages
date: 2025-12-09
categories:
- programming
tags:
- ruby
showToc: true
---

*Note*: adopted from Claude answer.

## Issue

If you are developing a ruby wrapper for a python package, the question is how to name this gem.


## **Use dashes: `ruby-package`**

This follows Ruby community standards and aligns with RubyGems conventions:

**Examples from the ecosystem:**
- `aws-sdk` (wraps AWS services)
- `google-cloud` (wraps Google Cloud APIs)
- `stripe-ruby` (Stripe's official Ruby wrapper)
- `twilio-ruby` (Twilio's official Ruby wrapper)

## Naming Patterns for Wrappers

**Pattern 1: `package-ruby`** (most common for official wrappers)
- Clear it's the Ruby version
- Example: `docling-ruby`

**Pattern 2: `ruby-package`** (common for community wrappers)
- Emphasizes it's a Ruby implementation
- Example: `ruby-docling`

**Pattern 3: Just `package`** (if name isn't taken)
- Cleanest, but often unavailable
- Example: `docling` (if available)

## Why Dashes, Not Underscores?

- **Gem names use dashes**: `my-gem` → installed as `my-gem`
- **Require paths use underscores**: `require 'my_gem'`
- RubyGems automatically converts dashes to underscores for requiring
- This is the established Ruby convention

## Module/Class Naming

Inside your gem, use `CamelCase`:

```ruby
# Gem: docling-ruby
# Require: require 'docling/ruby' or require 'docling_ruby'
# Module: Docling::Ruby or DocklingRuby
```

For a Python package wrapper, I'd go with **`docling-ruby`** or **`ruby-docling`** depending on whether it's official or community-maintained.
And since we are talking about a community (i.e. myself) maintained gem, I'm going with **`ruby-<package>`**.

