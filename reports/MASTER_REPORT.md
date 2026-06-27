# Portfolio Master Report — Aevum (Company X)

*AI-native autonomous-FTE showcase · point-in-time assessment of `/home/anjum/dev` as of 2026-06-26*

> **How to read this.** Every product was examined from its actual code, not its README
> claims. Each was scored **AI-native /5** (how much real AI vs. rules/stubs runs today)
> and **FTE-fit /5** (how naturally it maps to an autonomous "Digital FTE" that replaces a
> role). Maturity is **Idea → Prototype → MVP → Production**. The **honesty rule** is
> load-bearing: *built today* is kept strictly separate from *roadmap / AI gap*. Full
> per-product detail is in `projects/<slug>.md`; the scannable index is `PORTFOLIO.md`.

---

## 1. Executive summary

There are **25 assessed assets** spanning enterprise ops, fintech, security, CX, computer
vision, edtech, content, trading, and platform plumbing. The portfolio is **real** — not
slideware — but its headline marketing instinct (everything is "AI") **outruns what runs
today**. The single most important finding for a company whose entire pitch is *credibility
about AI*:

> **Most products that call themselves "AI" are today rule-based, keyword/regex, or a thin
> one-shot LLM passthrough. A minority are genuinely AI-woven. Almost none are *agentic*
> (planning + tool-use loops). And despite the "native AI" brand, the codebase runs mostly
> on Gemini, with Claude barely used.**

That is not a crisis — it is the **product roadmap and the company's reason to exist.** The
gap between "looks like an autonomous employee" and "is one" is exactly what Aevum sells.
Three assets already *embody* the autonomous-FTE thesis and should anchor the showcase;
the rest are an honest pipeline with a clear upgrade path to Digital FTEs.

**What's genuinely strong today**

- **One product is live in production** (`cms` / ComplaintHub, real customer, `$0` infra).
- **Two more are deployed MVPs** (`my-personal-assistant`, `card-bin-tracker`) and several
  are strong local MVPs (`opera-nova`, `vision-alert`, `hamdan-ent`).
- **The autonomous-FTE pattern is already real** in `opera-nova` (a literally-named
  "Auto-FTE" anomaly agent that flags costly reports out-of-band) and in `hackaton-0-fte`
  (NL→SQL→Excel finance reporting + inbox triage + WhatsApp/Xero/Gmail integration).
- **Deep, real AI** (multi-role prompting, RAG, provider-fallback) exists in
  `my-personal-assistant`, `my-personal-examiner`, and the `my-skills` capability library.
- **Two reusable platform assets** already power the rest: `whatsapp-mcp` (the 24/7
  conversational channel) and `my-skills` (71 vetted agent skills — the "hands").

**What holds it back (recurring across the portfolio)**

1. **AI is mostly shallow.** Rules dressed as AI (`email-spam-classifier`, `price-research`,
   `card-blocking-demo`, `a-level-learning`, `promptforge`), or single-shot LLM calls with
   no planning/tool-use. The agentic loop — the thing that makes a *Digital FTE* — is the
   common missing piece.
2. **Wrong/again-thin model strategy for an "AI-native" brand.** Recurrent Gemini multi-key
   rotation; OpenAI fallbacks; **Claude almost absent.** An AI-native company should
   standardize its flagship agents on the strongest reasoning models and own that choice.
3. **Security hygiene is a real liability.** Live API keys / OAuth tokens / DB credentials
   are **committed to the repo** in many projects (`my-personal-assistant`,
   `social-scrapper`, `my-personal-examiner`, `hackaton-0-fte`, `price-research`). This
   must be remediated before *anything* is shown to a customer or put on a public site.
4. **Single-tenant + hardcoded identity.** Owner phone, "Nando's Pakistan", named people,
   and VM IPs are baked in. No auth / multi-tenancy / billing / onboarding on most.
5. **Doc inflation.** Several status docs claim "98–100% complete / production-ready" while
   the code contradicts them (`a-level-learning`, parts of `hr-agent`, `promptforge`). The
   showcase must be built from verified functionality, never from these claims.

---

## 2. Portfolio matrix

Sorted by tier, then by maturity/scores. Scores are today's reality, not potential.

| # | Product | Category | Maturity | AI /5 | FTE /5 | Tier |
|--|---|---|---|:--:|:--:|---|
| 1 | **ComplaintHub** (`cms`) | CX / Complaint mgmt | **Production** | 2 | 4 | **Flagship** |
| 2 | **Personal AI Employee** (`hackaton-0-fte`) | Autonomous back-office | MVP | 3 | 4 | **Flagship** |
| 3 | **My Personal Assistant** (`my-personal-assistant`) | Inbox + delegation automation | MVP (deployed) | 4 | 4 | **Flagship** |
| 4 | **OperaNova** (`opera-nova`) | Restaurant/F&B back-office ERP | MVP (strong) | 3 | 4 | **Flagship** |
| 5 | **Vision Alert** (`vision-alert`) | Computer vision / physical security | MVP | 3 | 4 | **Flagship** |
| 6 | **My Personal Examiner** (`my-personal-examiner`) | EdTech (exam/tutor) | Prototype (adv.) | 4 | 4 | Secondary |
| 7 | **Hamdan ERP** (`hamdan-ent`) | B2B SaaS accounting/ERP | MVP | 1 | 4 | Secondary |
| 8 | **Sentinel CX** (`social-scrapper`) | CX / social listening | MVP (1 platform) | 3 | 3 | Secondary |
| 9 | **Phishing Scanner** (`phishing-scanner`) | Security | MVP | 2 | 3 | Secondary |
| 10 | **HR Agent** (`hr-agent`) | HR tech / AI recruiter | Prototype | 3 | 4 | Secondary |
| 11 | **Safe Journey Voice Agent** (`travel-insurance-voice-agent`) | Voice AI tele-sales | Prototype | 2 | 4 | Incubate |
| 12 | **Slides Creator** (`slides-creator`) | Content / productivity | Prototype | 1 | 4 | Incubate |
| 13 | **Uroojj POS/ERP** (`alyan-uroojj-pos`) | Retail ERP (Pakistan) | **Idea** (docs only) | 0.5 | 4 | Incubate |
| 14 | **SME Price Research Engine** (`price-research`) | Procurement / research | MVP | 1 | 2 | Incubate |
| 15 | **PromptForge** (`promptforge`) | AI productivity / tooling | MVP | 2 | 1 | Incubate |
| 16 | **Voice Card-Blocking** (`card-blocking-demo`) | Fintech / voice | Prototype | 2 | 3 | Incubate |
| 17 | **Deal Cracker** (`card-bin-tracker`) | Fintech (retail promos) | MVP (deployed) | 0 | 3 | Incubate |
| 18 | **Interactive Robotics Book** (`interactive-robotics-book`) | EdTech / publishing | Prototype | 3 | 2 | Incubate |
| 19 | **Cambridge AI Professor** (`a-level-learning`) | EdTech | Prototype (stalled) | 0.5 | 1.5 | Incubate |
| 20 | **Email Threat Classifier** (`email-spam-classifier`) | Security | Prototype | 0.5 | 2 | Incubate |
| 21 | **LinkedIn Post Writer** (`linkedin-post`) | Content / marketing | Prototype | 1 | 2 | Incubate |
| 22 | **Auto Trade System** (`auto-trade`) | Algo trading | MVP (forward-test) | 2 | 2 | Incubate |
| 23 | **Anjum AutoTrade** (`anjum-autotrade`) | Algo trading | Prototype (research) | 2 | 3 | Incubate |
| 24 | **WhatsApp MCP** (`whatsapp-mcp`) | Platform channel | MVP | 1 | 4 | **Platform** |
| 25 | **My Skills Collection** (`my-skills`) | Platform skill library | MVP | 4 | 5 | **Platform** |

---

## 3. Tiering & rationale

**Flagships (lead the showcase — proven + on-theme + demoable).**
`cms`, `hackaton-0-fte`, `my-personal-assistant`, `opera-nova`, `vision-alert`. These have
*working* substance and the clearest "this is an employee, not a tool" story. `cms` brings
production credibility; the other four bring the autonomous-FTE narrative with live demos.

**Secondary (strong, customer-ready with focused work).**
`my-personal-examiner` (deep AI, content-starved), `hamdan-ent` (excellent accounting
kernel, zero AI yet), `social-scrapper` (real AI core, 1 of 5 channels), `phishing-scanner`
(real ML + cost-gated LLM escalation), `hr-agent` (textbook AI-recruiter, blocked by model
bugs). Each is one well-scoped push from a flagship-grade demo.

**Incubate (honest works-in-progress / roadmap-heavy).** The remaining 11 vertical and
content products plus the two trading systems. Real ideas, thin or absent agentic AI, or
early maturity. Show as "in the lab" / coming-soon, not as headline proof.

**Platform capabilities (not sold standalone — the moat).** `whatsapp-mcp` (the channel)
and `my-skills` (the hands). These are reused across products and are a large part of why
Aevum can credibly ship Digital FTEs faster than competitors. Note `whatsapp-mcp` rides a
personal-account library (ToS/ban risk) — productizing means moving to the official
WhatsApp Business API.

---

## 4. Cross-cutting platform opportunity (the real company)

The portfolio keeps **re-implementing the same primitives** by hand. Consolidating them is
the highest-leverage move and the substance behind the "AI-native" brand:

- **A shared Agent Runtime** — a planning + tool-use loop with human-in-the-loop approval
  gates. Today every product hand-rolls "rules + one LLM call + an approval screen."
  `opera-nova`'s Auto-FTE agent and `hackaton-0-fte` are the closest existing templates.
- **An omnichannel I/O layer** — Gmail + WhatsApp (`whatsapp-mcp`) + voice (Whisper/TTS,
  already in `card-blocking-demo` / `travel-insurance-voice-agent`) + dashboards. Reused,
  not rebuilt per product.
- **The skills/capability library** (`my-skills`, 71 skills) as the agents' "hands."
- **A model-reliability layer** — the recurring Gemini multi-key rotation + fallback
  pattern, standardized; and a deliberate **flagship-on-Claude** policy for agentic
  reasoning to match the AI-native positioning.
- **Shared tenancy/auth/billing/audit** — every product needs the same multi-tenant,
  RBAC, audit-trail, and metering substrate. `cms` (RBAC + 100% audit) and `hamdan-ent`
  (Postgres RLS + immutable journal) already prove the patterns.

**Framed for the pitch:** Aevum isn't 25 apps — it's **one autonomous-FTE platform** with
25 role-shaped front ends (AI Complaints Officer, AI Back-Office Clerk, AI Recruiter, AI
Cost Controller, AI Security Guard, AI Examiner, AI Community Manager, AI Finance Clerk,
AI Voice Sales Rep…). That is the story the website should tell.

---

## 5. Recommended showcase lineup

1. **Hero / thesis:** "Hire the machine — autonomous FTEs that never sleep." One command-bar
   hero, the AI-spectrum aesthetic from `cms/marketing/`.
2. **Flagship pages (5), each told as a role it replaces, with a live demo hook:**
   - **ComplaintHub** → *AI Complaints Officer* (proof: live in production).
   - **Personal AI Employee** → *AI Back-Office Clerk* (demo: NL→finance report).
   - **My Personal Assistant** → *AI Chief-of-Staff* (demo: inbox triage + auto-delegation).
   - **OperaNova** → *AI Cost Controller* (demo: Auto-FTE flags a costly report).
   - **Vision Alert** → *AI Security Guard* (demo: live zone-intrusion alert to WhatsApp).
3. **"The platform" section:** the agent runtime + omnichannel + skills moat (§4).
4. **"In the lab" grid:** secondary + incubate products as coming-soon role cards.
5. **Trust/heritage band:** early-adopter AI track record; real customer (Nando's Pakistan);
   honest capability stats.

---

## 6. Honesty guidance for marketing (non-negotiable)

Carried from the `cms` marketing kit and reinforced by this audit:

- **Never present roadmap AI as shipped.** If a "reasoning"/"sentiment"/"agentic" feature is
  actually rules or a discarded LLM call (as in `hackaton-0-fte`'s dashboard,
  `card-blocking-demo`, `price-research`), do not demo it as AI. Show what runs.
- **Label aspirational numbers as targets.** No invented ROI or accuracy figures. Use real
  capability stats (channels, audited %, `$0` infra, "live in production").
- **Lead with the demo that works.** Each headline maps to a verified feature in the
  product's report, not its roadmap.
- **Fix secrets before publishing.** Rotate and remove all committed keys/tokens/credentials
  across the repos listed in §1.3 before any public showcase or customer demo.

---

## 7. Excluded from the catalog

Scratch/lab/personal/infra/superseded folders not assessed as products: `better-auth*`,
`prj_better-auth`, `better-test-auth`, `nextjs-cms` (superseded by `cms`), `openclaw*`,
`my_profile`, `nandos_helper`, `nandos_prj_visibility`, `nandos_projects`, `dsk_cleanup`,
`mikrotik`, `wifi_device_alert`, `zk`, `fastapi_code`, `full-Stack-todo`, `chico`,
`unreal_engine`, `sindh_boyscouts`, `claude-*` labs, `context_eng_skill_creator`,
`my-research-paper`, `tred_micro` (empty), `meal_minion` (empty). **Watchlist (incubate,
no deep report):** `evolution_to_do` / `arch_evol_to_do` — hackathon "evolution of todo"
builds; revisit if the AI-task-management angle is pursued.
