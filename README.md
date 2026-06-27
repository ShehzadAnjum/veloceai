# Aevum — The Restless Workforce (Company X)

> Working brand: **Aevum**. Private repository — contains candid internal portfolio
> analysis. Rename the brand freely.

An AI-native venture that equips corporates and industries with **autonomous FTEs** —
AI workers that own a job end-to-end and run 24/7, with humans in the loop only where it
counts. This repo holds the company vision, the portfolio analysis, and the showcase
portal built from it.

## Contents

| Path | What it is |
|---|---|
| [`CLAUDE.md`](CLAUDE.md) | Company vision, positioning, and working conventions (incl. the honesty rule). |
| [`reports/PORTFOLIO.md`](reports/PORTFOLIO.md) | Scannable catalog: 25 products tiered + scored (AI-native / FTE-fit). |
| [`reports/MASTER_REPORT.md`](reports/MASTER_REPORT.md) | Strategy: exec summary, matrix, tiering, platform opportunity, honesty guidance. |
| [`reports/projects/*.md`](reports/projects) | 25 per-project deep dives (built today vs. roadmap, AI gap, FTE upgrade). |
| [`site/`](site) | The showcase portal — landing, 5 flagship dossiers, and the lab page. |

## Run the showcase site

It's plain static HTML/CSS/JS — no build step. Serve from the repo root so the site's
links into `reports/` resolve:

```bash
python3 -m http.server 8055
# then open http://localhost:8055/site/index.html
```

## The five flagships

| Unit | Product | Maturity |
|---|---|---|
| AI Complaints Officer | ComplaintHub | **Live in production** |
| AI Back-Office Clerk | Personal AI Employee | MVP |
| AI Chief of Staff | Aria (Personal Assistant) | Deployed MVP |
| AI Cost Controller | OperaNova Auto-FTE | Strong MVP |
| AI Security Guard | Vision Alert | MVP |

Plus 20 more across security, fintech, edtech, voice, content and trading — see
[`site/lab.html`](site/lab.html) and the [portfolio](reports/PORTFOLIO.md).

---

*The site separates what runs **today** from what's on the **roadmap** on every page —
never present roadmap AI as shipped.*
