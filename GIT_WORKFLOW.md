# Git Workflow

## The One Rule Behind Everything
> If anyone else has seen your local commits → merge
> If only you have seen your local commits → rebase is safe

---

## Solo Feature Branch Workflow
```bash
# Start new work
git checkout -b feature/ex-1

# Work and commit as you go
git add <files>
git commit -m "description"

# First push — creates remote branch and links it
git push -u origin feature/ex-1

# Keep up to date with main (rebase safe — nobody has seen your local commits)
git pull --rebase origin main

# Update remote branch after rebase
git push --force-with-lease origin feature/ex-1

# When done — open PR on GitHub, review, merge via GitHub UI

# Sync local main after PR is merged
git checkout main
git pull
```

---

## Shared Feature Branch Workflow
```bash
# Start new work
git checkout -b feature/payments

# Work and commit as you go
git add <files>
git commit -m "description"

# First push — creates remote branch and links it
git push -u origin feature/payments

# Keep up to date with main (merge — main is shared)
git pull origin main

# Sync with teammate's latest work on same branch (merge — they may have seen your commits)
git pull origin feature/payments

# Push your changes (no force push needed — no rebase happened)
git push origin feature/payments

# When done — open PR on GitHub, review, merge via GitHub UI

# Sync local main after PR is merged
git checkout main
git pull
```

---

## Key Rules
- Never commit directly to main — always use a feature branch and PR
- Never force push to main or shared branches
- Never rebase/amend commits already pushed to a shared branch — just add a new commit
- Always merge into main via GitHub UI, never locally
- Use `--force-with-lease` instead of `--force` — safer, fails if someone else pushed since your last fetch
- One branch per exercise/feature — keeps history readable
