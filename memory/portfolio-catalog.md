---
name: portfolio-catalog
description: The 25-product assessment — tiers, the 5 flagships, scoring, and the cross-cutting honest findings.
metadata:
  type: reference
---

**What it is:** 25 projects from `/home/anjum/dev` examined **from source** (not READMEs) and
scored **AI-native /5** + **FTE-fit /5**, tiered Flagship / Secondary / Incubate / Park. Full
detail lives in `reports/`: `PORTFOLIO.md` (matrix), `MASTER_REPORT.md` (strategy),
`projects/<slug>.md` (one deep dive per product). The site renders from these.

**The 5 flagships (the showcase fleet):**
1. ComplaintHub (`cms`) — AI Complaints Officer — **Production, live** (nandospak.com/cms) — AI 2 / FTE 4.
2. Personal AI Employee (`hackaton-0-fte`) — AI Back-Office Clerk — MVP — AI 3 / FTE 4.
3. Aria / MPA (`my-personal-assistant`) — AI Chief of Staff — deployed MVP — AI 4 / FTE 4.
4. OperaNova (`opera-nova`) — AI Cost Controller (the "Auto-FTE" agent) — strong MVP — AI 3 / FTE 4.
5. Vision Alert (`vision-alert`) — AI Security Guard — MVP — AI 3 / FTE 4.

**Platform moat (reused, never sold standalone):** `my-skills` (71-skill library = the agents'
"hands", FTE 5) and `whatsapp-mcp` (the 24/7 conversational channel). The other ~13 sit in the
**lab** (field-ready / incubating).

**Cross-cutting honest findings (the company's reason to exist):**
- Most products that *call* themselves "AI" are today **rule-based / keyword / one-shot LLM**;
  few are agentic. Genuinely AI-deep: `my-personal-assistant`, `my-personal-examiner`, `my-skills`.
- The codebase runs mostly on **Gemini, with Claude barely used** — off-brand for "AI-native".
- **Committed secrets** (keys / OAuth / DB creds) exist in several *source* repos — must rotate
  before any public demo (MASTER_REPORT §6). They are **not** in this repo.
- **Doc inflation** in some repos ("98% complete" vs reality) — always verify from code.
