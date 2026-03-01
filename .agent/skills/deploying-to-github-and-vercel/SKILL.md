---
name: deploying-to-github-and-vercel
description: Deploys a static or simple HTML/JS/CSS website to a GitHub repository and auto-triggers a Vercel production deployment. Use this skill when the user asks to "upload to GitHub", "deploy to Vercel", "push the site", or "go live". Handles first-time repo initialization, .gitignore creation, committing all project files, pushing to the remote branch, and verifying Vercel deployment status via the Vercel MCP. Assumes the Vercel project is already linked to the GitHub repo in the Vercel dashboard.
---

# Deploying a Website to GitHub + Vercel

## When to use this skill
- User says "upload to GitHub", "push to repo", "deploy to Vercel", "go live", "make it live"
- The project is a static site (HTML/CSS/JS) with local files ready to publish
- The Vercel project already exists and is linked to the GitHub repository
- First-time commit **or** pushing new changes to an existing repo

---

## Pre-flight Checklist
Copy and track this checklist at the start of each deployment:

- [ ] Identify project root directory
- [ ] Confirm target GitHub repo URL from the user
- [ ] Check `git status` — is this a fresh repo or existing?
- [ ] Create / verify `.gitignore`
- [ ] Stage all relevant files (exclude OS artifacts)
- [ ] Commit with a descriptive message
- [ ] Push to remote (set upstream if first push)
- [ ] Confirm Vercel auto-deployment triggered and reached READY state

---

## Workflow

### Step 1 — Check Git State
```bash
git status
git remote -v
git log --oneline -5
```
- If `No commits yet` → first-time setup, follow Step 2A
- If commits exist → skip to Step 2B (push changes)
- If remote is missing → add it: `git remote add origin <repo-url>`
- If remote already exists with wrong URL → `git remote set-url origin <repo-url>`

---

### Step 2A — First-Time Setup (No commits yet)

**Always create a `.gitignore` first** to exclude Windows/OS artifacts:

```
nul
*.log
.DS_Store
Thumbs.db
desktop.ini
```

Then stage all real project files explicitly — **never use `git add .`** blindly on Windows because the `nul` file and other OS artifacts will slip in:

```bash
git add .gitignore index.html <folder1>/ <folder2>/
git status   # Verify staged files look correct
```

Commit and push, setting upstream:
```bash
git commit -m "Initial commit: <Project Name>"
git push -u origin master
```

> ⚠️ Large media files (images, videos, animation frames) will cause the push to take longer. This is normal. The push runs in background — monitor with `command_status`.

---

### Step 2B — Pushing Updates (Existing Repo)

```bash
git add -A
git status   # Verify what's staged
git commit -m "<descriptive message about what changed>"
git push
```

---

### Step 3 — Verify Vercel Deployment

After the GitHub push, Vercel auto-deploys **if the project is linked**. Verify using the Vercel MCP:

```
mcp_vercel-mcp_vercel_list_deployments(projectId: "<project-id>", limit: 3)
```

Check `readyState` in the response:
- `BUILDING` → still deploying, wait ~30 seconds and check again
- `READY` + `readySubstate: PROMOTED` → ✅ live in production
- `ERROR` → see [Troubleshooting](#troubleshooting)

Use `mcp_vercel-mcp_vercel_list_projects()` first if you don't know the `projectId`.

---

### Step 4 — Report to User

Always report back with:
1. ✅ GitHub repo URL
2. ✅ Vercel live URL (from `alias` array in deployment response)
3. ✅ Branch-specific preview URL (if applicable)
4. 💡 Reminder that future `git push` auto-redeploys

---

## Key Rules & Gotchas

- **Windows `nul` file**: Windows sometimes creates a `nul` file in project folders. Always add it to `.gitignore` before staging.
- **Large binary files**: 240+ images in `hero_animation/` type folders are fine to commit — just expect slow push times (1-3 min).
- **Production branch**: Vercel's auto-deploy only triggers on the branch set as "Production Branch" in Vercel (usually `main` or `master`). Confirm with `link.productionBranch` in the project data.
- **Branch mismatch**: If local branch is `master` but Vercel expects `main`, either rename the branch or update Vercel's production branch setting.
- **Remote already exists error**: `error: remote origin already exists` is safe to ignore — just verify the URL with `git remote -v`.
- **SSO protection**: New Vercel deployments may have SSO enabled by default. If the URL shows a login wall, disable it in Vercel dashboard under Project Settings → Deployment Protection.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `remote origin already exists` | Run `git remote -v` to verify URL, ignore the error if correct |
| Push rejected (non-fast-forward) | Run `git pull --rebase origin master` then push again |
| Vercel state stays `BUILDING` | Wait 60s, re-check with `list_deployments`. If stuck, use `vercel_redeploy` |
| Vercel state is `ERROR` | Check `inspectorUrl` from deployment data to see build logs |
| Site shows old version | Hard-refresh browser (Ctrl+Shift+R) — Vercel CDN may be cached |
| Auth popup on Vercel URL | Disable SSO in Vercel → Project Settings → Deployment Protection |

---

## Resources
- [scripts/check-deployment.md](scripts/check-deployment.md) — MCP query reference for checking deployment status
- [examples/gitignore-template.txt](examples/gitignore-template.txt) — Standard `.gitignore` for static sites
