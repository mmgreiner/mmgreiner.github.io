---
title: Changelog in git
description: Tools to automatically create the changelog file in github
date: 2026-02-12
draft: true
categories:
- programming
- publishing
tags:
- git
- github
showToc: true
---

There are a few solid approaches depending on how much control you want:

## Using a dedicated tool (recommended)

**conventional-changelog** is the most popular option. It works best if your commits follow the [Conventional Commits](https://www.conventionalcommits.org/) format (`feat:`, `fix:`, `chore:`, etc.).

```bash
npm install -g conventional-changelog-cli
conventional-changelog -p angular -i CHANGELOG.md -s
```

The `-s` flag writes back to the same file, and `-p angular` uses the Angular commit convention (most common). Run it whenever you want to update the changelog.

**release-please** (by Google) goes further — it opens a PR with a generated CHANGELOG and bumps your version automatically. Great for CI pipelines.

---

## Using git-cliff (simpler, Rust-based)

[git-cliff](https://git-cliff.org/) is fast and highly configurable via a `cliff.toml` file:

```bash
# Install
cargo install git-cliff
# or: brew install git-cliff

# Generate changelog
git-cliff -o CHANGELOG.md
```

It works even without strict Conventional Commits formatting, and the config lets you customize groupings, output templates, and filtering.

---

## Pure git, no tools

If you want something lightweight with no dependencies:

```bash
git log --pretty=format:"- %s (%h)" --no-merges > CHANGELOG.md
```

You can get fancier with tags to group by version:
```bash
git log v1.0.0..HEAD --pretty=format:"- %s (%h)" --no-merges
```

---

## Automating it in CI

Most teams run changelog generation as part of their release workflow. With GitHub Actions:

```yaml
- name: Generate Changelog
  run: npx conventional-changelog-cli -p angular -i CHANGELOG.md -s

- name: Commit changelog
  run: |
    git config user.name "github-actions"
    git add CHANGELOG.md
    git commit -m "chore: update changelog"
```

---

**My recommendation:** if you're starting fresh, adopt Conventional Commits + `git-cliff`. It's the least opinionated setup and produces clean, readable changelogs with minimal config. If you're already on npm/Node, `conventional-changelog` has the biggest ecosystem.
