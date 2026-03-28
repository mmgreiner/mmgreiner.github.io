Run git-cliff **first**, then tag.

The typical workflow is:

```bash
# 1. Generate/update the changelog (bumps to next version automatically)
git cliff --bump -o CHANGELOG.md

# 2. Stage and commit it
git add CHANGELOG.md
git commit -m "chore(release): prepare for v1.2.0"

# 3. Then tag
git tag v1.2.0

# 4. Push both
git push && git push --tags
```

**Why this order?**

- `--bump` lets git-cliff calculate the next version from your commits automatically (based on conventional commit types), so you don't have to decide the version number manually
- The changelog commit itself is skipped in future runs because of your existing rule: `{ message = "^chore\\(release\\): prepare for", skip = true }` — which you already have set up perfectly
- If you tagged first, that release commit would appear in the *next* tag's changelog, which is usually not what you want

If you already know the version and don't want auto-bump, you can also do:

```bash
git cliff --tag v1.2.0 -o CHANGELOG.md
```

Same order applies — changelog commit, then tag.Based on your `cliff.toml`, your commit prefixes that get sorted into groups are:

| Prefix | Group |
|---|---|
| `feat` | 🚀 Features |
| `fix` | 🐛 Bug Fixes |
| `refactor` | 🚜 Refactor |
| `doc` | 📚 Documentation |
| `perf` | ⚡ Performance |
| `style` | 🎨 Styling |
| `test` | 🧪 Testing |
| `chore` / `ci` | ⚙️ Miscellaneous Tasks |
| `revert` | ◀️ Revert |
| anything else | 💼 Other |

**Skipped** (won't appear in changelog at all):
- `chore(release): prepare for ...`
- `chore(deps...)` 
- `chore(pr)`
- `chore(pull)`
- commits matching `body = ".*security"` → 🛡️ Security *(body match, not prefix)*

**Usage example:**
```
feat: add login page
fix(auth): resolve token expiry bug
refactor(api): simplify request handler
chore: update dependencies
```