# Aevum — Autonomous-FTE Showcase Portal  (working name; rename freely)

This repo is the **showcase website + portfolio analysis** for an AI-native venture that
sells **autonomous FTEs** — AI agents that own a job end-to-end and run 24/7, human-in-the-
loop where it counts. The brand **"Aevum / The Restless Workforce"** is a *proposal*; treat
the name as a placeholder until locked. Full positioning: `memory/identity-and-positioning.md`.

## Non-negotiable rules
- **Honesty rule (load-bearing):** never present roadmap AI as shipped. Every product page and
  report splits *what runs today* from *roadmap / AI gap*; label aspirational numbers as
  targets; only demo what's real. The whole pitch is credibility about AI.
- **Smallest viable diff** — don't refactor unrelated things.
- **No secrets in the repo** — `.env` + docs only; never commit tokens/keys.
- **`reports/` is the source of truth** for the product catalog; the site renders from it.

## Layout
- `CLAUDE.md` — this file.  ·  `MEMORY.md` — index of the memory store (**read it first**).
- `memory/` — durable project notes, one topic per file.
- `reports/` — `PORTFOLIO.md` (matrix) · `MASTER_REPORT.md` (strategy) · `projects/<slug>.md` (25 deep dives).
- `site/` — the live showcase: `index.html`, `lab.html`, `products/*.html`, `assets/`. Static, no build.

## Live facts
- **GitHub:** `ShehzadAnjum/company_x` (**private**). Push to `main`; end every commit with the
  `Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>` trailer. Collaborator
  + conventions: `memory/repo-and-access.md`.
- **Run the site:** from the repo **root** → `python3 -m http.server 8055` →
  http://localhost:8055/site/index.html  (serve from root so the lab's `../reports/` links resolve).
- **Design system:** the CMS enterprise "AI spectrum" kit (`/home/anjum/dev/cms/marketing/`),
  ported into `site/assets/styles.css`. Details + rebrand tokens: `memory/showcase-site.md`.

## Start here each session
1. Read `MEMORY.md` and the relevant `memory/*.md` before acting — work done, decisions, and the
   open TODO list live there. Don't answer from assumption.
2. For anything about a specific product, read its `reports/projects/<slug>.md`.
3. When something durable changes, update the right `memory/` file (and `MEMORY.md` if new);
   keep this file lean (< 60 lines).
