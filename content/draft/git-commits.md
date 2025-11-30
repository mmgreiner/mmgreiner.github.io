
## Using conventions in Github commits

Taken from ChatGPT

Absolutely — **`conventional-changelog`** is a great tool to understand if you care about **automatic changelog generation** and **consistent commit messages**.

Let’s go step by step 👇

---

## 🧩 What is `conventional-changelog`?

**`conventional-changelog`** is a Node.js tool (and family of libraries) that automatically generates changelogs from **commit messages that follow the [Conventional Commits](https://www.conventionalcommits.org/) specification**.

It was originally built by the Angular team and is now widely used in many ecosystems.

---

## 🧠 What are “Conventional Commits”?

They’re commit messages that follow a simple, structured convention:

```
<type>(<optional scope>): <short summary>

[optional body]

[optional footer]
```

### Example:

```
feat(auth): add login with Google
fix(api): handle 500 errors properly
chore(deps): update Rails to 7.1
```

### Common types:

* `feat`: A new feature
* `fix`: A bug fix
* `chore`: Maintenance or tooling changes
* `docs`: Documentation updates
* `refactor`: Code restructuring
* `test`: Adding or updating tests
* `perf`: Performance improvements

This structure allows tools to **parse commits automatically**.

---

## ⚙️ How `conventional-changelog` works

It parses your git history and generates a changelog (typically a `CHANGELOG.md` file), grouping commits by type and optionally bumping versions automatically.

### Example output:

```markdown
# Changelog

## [1.2.0] - 2025-10-22

### 🚀 Features
- Add login with Google (#45)

### 🐛 Bug Fixes
- Handle 500 errors properly (#48)

### 🧰 Maintenance
- Update Rails to 7.1 (#50)
```

---

## 🔧 How to use it

### 1. Install it

```bash
npm install -g conventional-changelog-cli
```

### 2. Generate a changelog

From your project root:

```bash
conventional-changelog -p angular -i CHANGELOG.md -s
```

* `-p angular` uses the Angular-style convention (most common).
* `-i CHANGELOG.md` points to your changelog file.
* `-s` means “in place” (updates the file).

Run this command each time you release.

---

## 🪄 Automating it

You can add a script to your `package.json`:

```json
"scripts": {
  "changelog": "conventional-changelog -p angular -i CHANGELOG.md -s"
}
```

Then just run:

```bash
npm run changelog
```

---

## ⚡ Related Tools in the Same Ecosystem

| Tool                            | Purpose                                                     |
| ------------------------------- | ----------------------------------------------------------- |
| **conventional-commits-parser** | Parses conventional commit messages                         |
| **conventional-changelog**      | Generates changelogs                                        |
| **standard-version**            | Automates version bumping + changelog                       |
| **semantic-release**            | Fully automates release + changelog + tagging on GitHub/npm |

---

## 🧮 Comparison with Similar Tools

| Tool                       | Based on Commits | Based on PRs | Auto version bump | Auto release to GitHub/npm |
| -------------------------- | ---------------- | ------------ | ----------------- | -------------------------- |
| **conventional-changelog** | ✅ Yes            | ❌            | ❌                 | ❌                          |
| **standard-version**       | ✅ Yes            | ❌            | ✅                 | ❌                          |
| **semantic-release**       | ✅ Yes            | ❌            | ✅                 | ✅                          |
| **release-drafter**        | ❌                | ✅            | ❌                 | 🟡 Drafts releases         |

So:

* Use **`conventional-changelog`** if you just want to **generate changelogs**.
* Use **`standard-version`** if you want **changelog + version bumping**.
* Use **`semantic-release`** if you want **fully automated publishing**.

---

Would you like me to show how to integrate `conventional-changelog` (or `standard-version`) into a **Rails project’s release workflow**, even though it’s not a Node app? (It’s possible and works well with CI/CD setups.)

