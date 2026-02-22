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