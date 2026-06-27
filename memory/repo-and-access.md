---
name: repo-and-access
description: GitHub repo, visibility, the Hamza collaborator, and commit/push conventions for company_x.
metadata:
  type: reference
---

**Repo:** `github.com/ShehzadAnjum/company_x` — **private**. Default branch `main`.
Owner/admin: ShehzadAnjum (git identity *Shehzad Anjum <sanjum77@gmail.com>*).

**Collaborator — Hamza:** GitHub username **`0hamzashehzad0-commits`**, role **Write** (can
clone, push branches, and open PRs directly — no fork needed). Invited 2026-06-26 by email
(`0hamza.shehzad@gmail.com`); **the invite expires 2026-07-03** if unaccepted. Check status:
`gh api /repos/ShehzadAnjum/company_x/invitations` (pending) and `.../collaborators` (accepted).

**Conventions:**
- Pushing directly to `main` is the current norm (solo owner). End **every** commit with:
  `Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>`.
- `.gitignore` guards `.env` / `*.key` / `*.pem` / logs. Never commit secrets.
- **Branch protection: NOT enabled** — requires **GitHub Pro** for a private repo (free plan
  returns 403 for both classic protection and rulesets). **Decision 2026-06-27:** leave `main`
  open / direct pushes. To enable later: upgrade to Pro, or make a public site-only repo.
- Public-site split is an open option (host only `site/`, keep `reports/` private) — [[worklog-and-todo]].
