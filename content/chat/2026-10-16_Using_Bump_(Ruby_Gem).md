---
title: 🤖 Using Bump (Ruby Gem)
description: A Ruby gem that updates the version of your gem or Ruby project, commits the change, and can optionally tag the release.
date: 2026-10-16
draft: false
categories: 
- programming
tags:
- ruby
showToc: false
---

# Using `bump` (Ruby gem)

[bump] updates the version of your gem or Ruby project, commits the change, and can optionally tag the release.

[bump]: https://github.com/gregorym/bump

## Install

```bash
gem install bump
```

Or add it to your `Gemfile`:

```ruby
group :development do
  gem "bump", require: false
end
```

Or add it from the command line:

```bash
bundle add bump --group development
```

## Basic commands

Run these from the root of your project (where the `.gemspec` or `VERSION` file lives).

```bash
bump current          # show the current version
bump patch            # 1.2.3 -> 1.2.4
bump minor            # 1.2.3 -> 1.3.0
bump major            # 1.2.3 -> 2.0.0
bump pre              # 1.2.3 -> 1.2.3.pre, then 1.2.3.pre2, ...
bump set 2.0.0        # set an exact version
bump show-next patch  # preview the next version without changing anything
```

## Common options

```bash
bump patch --tag                 # also create a git tag (e.g. v1.2.4)
bump patch --tag-prefix release- # custom tag prefix
bump patch --no-commit           # change the version file only, no git commit
bump patch --no-bundle           # skip running `bundle` to update Gemfile.lock
bump patch --commit-message "Release"  # custom commit message
bump patch --changelog           # also update CHANGELOG.md
bump patch --replace-in README.md      # replace the old version in other files
```

Run `bump help` to see everything available in your installed version.

## Typical release flow

```bash
git status            # make sure your working tree is clean
bump minor --tag      # bump, commit, and tag
git push --follow-tags
gem build *.gemspec && gem push *.gem
```

## Rake tasks (optional)

Add to your `Rakefile`:

```ruby
require "bump/tasks"
```

Then use:

```bash
rake bump:patch
rake bump:minor
rake bump:major
rake bump:current
```

## Notes

- `bump` looks for the version in `lib/**/version.rb`, a `VERSION` file, or your `.gemspec`.
- Commit or stash unrelated changes first, since `bump` creates a commit for you.

