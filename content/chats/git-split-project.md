Yes! `git subtree` is actually a great approach for this — it's built into git (no extra tools needed) and is a bit more straightforward than `filter-repo`.

---

**Extracting a subfolder using git subtree**

1. **In your existing monorepo**, split out the subfolder into a new local branch:
   ```bash
   git subtree split --prefix=your/subfolder/path -b my-new-branch
   ```
   This creates a new branch containing only the commits that touched that subfolder, with the subfolder contents at the root.

2. **Create a new empty repo in Azure DevOps** (no README, no .gitignore — keep it completely empty).

3. **Push the new branch to the new repo:**
   ```bash
   git push https://your-org@dev.azure.com/your-org/new-repo/_git/new-repo my-new-branch:main
   ```

4. **Clean up the local branch** when done:
   ```bash
   git branch -d my-new-branch
   ```

---

**Optionally remove the subfolder from the monorepo**

```bash
git rm -r your/subfolder/path
git commit -m "Remove subfolder — extracted to its own repo"
git push
```

---

**How it compares to filter-repo**

| | `git subtree split` | `git filter-repo` |
|---|---|---|
| Built into git | ✅ Yes | ❌ Needs install |
| Preserves history | ✅ Yes | ✅ Yes |
| Works on current clone | ✅ Yes | ⚠️ Needs fresh clone |
| Speed on large repos | 🐢 Can be slow | ⚡ Much faster |
| Handles tags | ❌ No | ✅ Yes |

So `git subtree` is more convenient for smaller repos, but if your monorepo is large or has a long history, `filter-repo` will be significantly faster.