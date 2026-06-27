# Aevum Portfolio — Index

*The catalog the showcase portal is built from. 25 assessed products + 2 platform
capabilities, examined from source on 2026-06-26. Each links to a full deep-dive in
`projects/<slug>.md`. Strategy, tiering, and cross-cutting findings live in
[`MASTER_REPORT.md`](./MASTER_REPORT.md). Company vision: [`../CLAUDE.md`](../CLAUDE.md).*

**Legend** — **AI** = real AI running today /5 · **FTE** = autonomous-FTE fit /5 ·
Maturity = Idea → Prototype → MVP → Production. Scores are today's reality, not potential.
**Honesty rule:** *built today* is separated from *roadmap* in every report.

---

## ★ Flagships — lead the showcase (proven + on-theme + demoable)

| Product | Role it sells as | Maturity | AI | FTE | Hook |
|---|---|---|:--:|:--:|---|
| [ComplaintHub](./projects/cms.md) | **AI Complaints Officer** | Production | 2 | 4 | Omnichannel complaint system of record, live in production, $0 infra |
| [Personal AI Employee](./projects/hackaton-0-fte.md) | **AI Back-Office Clerk** | MVP | 3 | 4 | A 24/7 AI employee that triages inboxes and writes finance reports |
| [My Personal Assistant](./projects/my-personal-assistant.md) | **AI Chief-of-Staff** | MVP (deployed) | 4 | 4 | The teammate who never sleeps on your inbox and delegations |
| [OperaNova](./projects/opera-nova.md) | **AI Cost Controller** | MVP (strong) | 3 | 4 | The back-office employee who already read every report and flagged the costly ones |
| [Vision Alert](./projects/vision-alert.md) | **AI Security Guard** | MVP | 3 | 4 | Turn the NVR you already own into a 24/7 AI guard |

## ◆ Secondary — strong, customer-ready with focused work

| Product | Role it sells as | Maturity | AI | FTE | Hook |
|---|---|---|:--:|:--:|---|
| [My Personal Examiner](./projects/my-personal-examiner.md) | **AI Examiner / Tutor** | Prototype (adv.) | 4 | 4 | An A* Cambridge examiner-tutor that marks and coaches 24/7 |
| [Hamdan ERP](./projects/hamdan-ent.md) | **AI Finance Clerk** | MVP | 1 | 4 | Rupee-accurate double-entry kernel ready for autonomous AI finance agents |
| [Sentinel CX](./projects/social-scrapper.md) | **AI Community Manager** | MVP (1 platform) | 3 | 3 | Always-on AI that reads, scores, and drafts replies to every review |
| [Phishing Scanner](./projects/phishing-scanner.md) | **AI SOC Analyst** | MVP | 2 | 3 | Explainable phishing triage with cost-gated LLM escalation, 24/7 |
| [HR Agent](./projects/hr-agent.md) | **AI Recruiter** | Prototype | 3 | 4 | CVs screened, scored, and ranked 24/7 before you wake up |

## ○ Incubate — honest works-in-progress / roadmap-heavy ("in the lab")

| Product | Role it sells as | Maturity | AI | FTE | Hook |
|---|---|---|:--:|:--:|---|
| [Safe Journey Voice Agent](./projects/travel-insurance-voice-agent.md) | **AI Voice Sales Rep** | Prototype | 2 | 4 | Always-on voice tele-sales rep that quotes, qualifies, and captures leads |
| [Slides Creator](./projects/slides-creator.md) | **AI Presentation Designer** | Prototype | 1 | 4 | Brief in, branded editable PowerPoint deck out by morning |
| [Uroojj POS/ERP](./projects/alyan-uroojj-pos.md) | **AI Retail Back-Office** | Idea (docs only) | 0.5 | 4 | Legacy Access perfume ERP rebuilt as AI-run bilingual back-office |
| [SME Price Research Engine](./projects/price-research.md) | **AI Procurement Analyst** | MVP | 1 | 2 | Plain-language need in, sourced ranked PKR top-5 out |
| [PromptForge](./projects/promptforge.md) | **AI Workflow Builder** | MVP | 2 | 1 | Turn a prompt template into a self-running AI worker |
| [Voice Card-Blocking](./projects/card-blocking-demo.md) | **AI Bank Voice Agent** | Prototype | 2 | 3 | Block a stolen card by voice, Urdu or English, 24/7 |
| [Deal Cracker](./projects/card-bin-tracker.md) | **AI Promo Ops Agent** | MVP (deployed) | 0 | 3 | Type a card BIN, staff instantly see the right discount |
| [Interactive Robotics Book](./projects/interactive-robotics-book.md) | **AI Author + Tutor** | Prototype | 3 | 2 | Autonomous AI authors the chapters, 24/7 AI tutor teaches |
| [Cambridge AI Professor](./projects/a-level-learning.md) | **AI Marker** | Prototype (stalled) | 0.5 | 1.5 | A-Level marking agent grounded in real Cambridge mark schemes |
| [Email Threat Classifier](./projects/email-spam-classifier.md) | **AI Email Screener** | Prototype | 0.5 | 2 | Rules-only Gmail CLI today; clear path to an AI email screener |
| [LinkedIn Post Writer](./projects/linkedin-post.md) | **AI Content Marketer** | Prototype | 1 | 2 | Always-on LinkedIn ghostwriter: on-brand posts, on schedule |
| [Auto Trade System](./projects/auto-trade.md) | **AI Trade Filter** | MVP (forward-test) | 2 | 2 | AI co-pilot that filters every Gold signal, shows its work |
| [Anjum AutoTrade](./projects/anjum-autotrade.md) | **AI Risk Officer** | Prototype (research) | 2 | 3 | The AI risk officer that enforces a trading rulebook 24/7 |

## ⬡ Platform capabilities — the moat (reused, not sold standalone)

| Capability | Role | Maturity | AI | FTE | Hook |
|---|---|---|:--:|:--:|---|
| [My Skills Collection](./projects/my-skills.md) | **The agents' hands** (71 skills) | MVP | 4 | 5 | The shared skills library that gives every FTE real hands |
| [WhatsApp MCP](./projects/whatsapp-mcp.md) | **The agents' voice** (channel) | MVP | 1 | 4 | The WhatsApp channel our autonomous FTEs read and reply through 24/7 |

---

## Snapshot stats

- **25 products assessed** + **2 platform capabilities** — full reports in `projects/`.
- **Maturity:** 1 Production · 9 MVP (3 deployed) · 13 Prototype · 1 Idea · (2 platform MVP).
- **AI-native today:** 3 score 4/5 (`my-personal-assistant`, `my-personal-examiner`,
  `my-skills`); most score 1–3 — real AI is the roadmap, which is the company's reason to exist.
- **Top FTE-fit:** `my-skills` (5), then a cluster of 4/5 across the flagships + ERPs.
- **Cross-cutting:** consolidate one **Agent Runtime + omnichannel + skills** platform
  rather than 25 hand-rolled "rules + one LLM call" stacks. See [`MASTER_REPORT.md` §4](./MASTER_REPORT.md).

> **Before publishing anything:** rotate/remove committed secrets (keys, OAuth tokens, DB
> creds) found in several repos, and never present roadmap AI as shipped. See
> [`MASTER_REPORT.md` §6](./MASTER_REPORT.md).
