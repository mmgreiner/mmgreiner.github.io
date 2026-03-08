---
title: github, azure and mirrors
description: A chat about github and azure repositories using mirrors
date: 2026-01-15
draft: false
categories:
- programming
tags:
- git
- azure
- github
showToc: false
---

# Simple mirror push

Simple mirror push — just a few commands:

**1. Clone the GitHub repo as a bare clone**
```bash
git clone --bare https://github.com/your-username/your-repo.git
cd your-repo.git
```
A bare clone captures everything — all branches, tags, and history.

**2. Create a new empty repo in Azure DevOps**
Go to your Azure DevOps project → Repos → New repository. Leave it completely empty (no README).

**3. Mirror-push to Azure**
```bash
git push --mirror https://your-org@dev.azure.com/your-org/your-project/_git/your-repo
```
`--mirror` pushes all branches, tags, and refs in one shot.

**4. Clean up the bare clone**
```bash
cd ..
rm -rf your-repo.git
```

**5. Re-clone fresh from Azure** (optional but recommended)
```bash
git clone https://your-org@dev.azure.com/your-org/your-project/_git/your-repo
```

---

That's it. Your Azure repo will be an exact copy of the GitHub one with full history.

**A couple of things to sort out afterwards:**
- Update any CI/CD pipelines that pointed to GitHub
- Let collaborators know to update their remotes: `git remote set-url origin <new-azure-url>`
- If you want to keep GitHub as a read-only mirror or archive, you can leave it as-is or delete/archive it


# Using subtree

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