# OperaNova — a reverse-engineered restaurant back-office portal with a bolt-on autonomous "Auto-FTE" agent

## 1. Snapshot
- **One-liner:** A modern, normalized rebuild of a legacy multi-tenant restaurant admin portal (menu, inventory, accounting, loyalty, HR, reporting) — now carrying two real AI features: a multi-persona Recipe Advisor and an autonomous "Auto-FTE" anomaly-monitoring agent.
- **Category:** Vertical SaaS — restaurant/F&B back-office ERP (POS-adjacent admin), with an emerging agentic-ops layer.
- **Target buyer:** Multi-location restaurant chains / franchise operators (the seed tenant is "Nandos Pakistan"); buyers are the food-cost controller, ops manager, and finance/inventory teams.
- **Tech stack:** Next.js 16 (App Router) · React 19 · TypeScript · Tailwind v4 · shadcn/radix · Drizzle ORM · Postgres 16 (Docker, port 5544) · Better Auth (username RBAC) · TanStack Query/Table · Zod · Recharts · nodemailer. AI: OpenAI Chat Completions (GPT-4o/4.1/5) + Google Gemini (2.0/2.5/3) with key-rotation fallback; no Anthropic SDK wired (mentioned only in a comment). Python scripts (`scripts/*.py`) for SOP/costing extraction. Evidence: `clone/package.json`, `clone/src/lib/ai/catalog.ts`.
- **Maturity:** **MVP (strong, leaning toward Production for the CRUD core).** Justification: 113 page routes, 37 Drizzle migrations, real-data ETL from the live legacy portal (60 suppliers / 331 ingredients / 223 products / 150 customers), DB-driven RBAC (~218 permission keys × 4 actions), CI workflow, and per-module integration tests. The two AI features are working and integration-test-verified but **uncommitted** (sit on top of commit `f68d1a4`) and **not reflected in the authoritative `tasks/CHECKPOINT.md`** (which still ends at the 2026-06-12 rebrand). No Playwright E2E harness exists; verification is `tsx` DB integration tests.
- **Live URL / demo:** None public. Local only: `docker start operanova-pg`; in `clone/` load `.env` → `npm run dev` → `http://localhost:3000`, login `admin_nandos` / `Nandos@.123!`.
- **Path:** /home/anjum/dev/opera_nova

## 2. What's built today (verified)
**Core admin portal (classic CRUD, no AI) — feature-complete vs the legacy original:**
- **Menu:** Categories, Products (+ Size×Flavor price-point matrix), Variants, Addons (+ options/chaining), Timings, Recipes (with decimal portions), and a **Recipe Change-Control module** (`/menu/recipe-changes`): propose → verify → approve state machine with enforced segregation-of-duties, per-stage email-file attachments stored as Postgres `bytea`, and an immutable `recipe_versions` evolution timeline. Evidence: `clone/src/server/menu/recipe-change.*`, `tasks/CHECKPOINT.md` Phase 4.
- **Inventory:** Suppliers (+contacts/bank accounts), Ingredients (unit-conversion triple, reorder level), Item Categories, Measurements, plus 6 workflow documents on a header+line-items + approve/cancel state machine that posts to a real append-only **stock ledger** (`stock_movements` + materialized `stock_levels`): Purchase Orders, Transfers, Gate Passes, Sale Orders, Purchase Returns, Demand Notes, Prep Sheets, Stock Evaluations, Wastage. Cancel reverses stock idempotently. Evidence: `tasks/CHECKPOINT.md` §(c) + Waves 0/3.
- **Orders/POS sales subsystem:** orders + line items with recipe-driven stock deduction on completion and reversal on void.
- **Accounting:** Chart of Accounts (sub-account hierarchy), Expenses (+ heads/sub-heads), 5-way typed double-entry vouchers (approve/unapprove), Account Taxes, and 5 computed reports (Ledger, Trial Balance, Balance Sheet, P&L) that tie out.
- **Loyalty:** all 8 promotion types (Coupons, Discount Programs, Vouchers, Reward Points, Deals, Comps, POS Discounts, Automatic Discounts) with channel/location/product scoping.
- **Reports:** 15+ Recharts-backed reports with reusable CSV export. **HR, Customers, Locations:** staff master (departments, provisioning, terminate/reinstate), customer addresses + loyalty enrollment + spend aggregates, location operating-hours/geo.
- **Settings (12/12):** permission-groups matrix editor, taxes, payment methods, delivery rates, POS receipts, operational toggles, billing+invoices, printers, 3rd-party, notifications.
- **Foundations:** Better Auth username login, DB-driven RBAC + `proxy.ts` gate + RBAC-filtered sidebar, audit log on every mutation, isolated Postgres, CI (`.github/workflows/ci.yml`).

**AI feature #1 — Recipe Expert Advisor (working):** a two-persona chat at `/menu/recipe-advisor` — "Recipe Perfectionist" (chef) and "Food Cost Controller" (finance) — with **agentic cross-consultation**: the active bot can call a `consult_specialist` tool, the other persona answers, and the first streams a final answer incorporating it (`clone/src/server/ai/advisor-agent.ts`). Streaming, provider-agnostic OpenAI/Gemini with transient-failure fallback (`providers.ts`). Strong **RAG-style grounding**: each turn's system prompt is built from the selected recipe's real BoM (with prep-batch expansion), per-channel costing pulled from the "Proposed Pricing" workbook, 12-month sales volume, extracted SOP knowledge, and optional `knowledge/*.md` reference docs (`personas.ts`, `recipe-advisor.ts`). Per-bot toggles (allow-swap, cost-optimization, replacement, target cost). RBAC-gated on `recipe_change_control`.

**AI feature #2 — Auto-FTE anomaly agent (working, the headline asset):** `clone/src/server/fte/agent.ts` — a genuinely autonomous monitor with **5 detectors**: (1) wrong-unit/over-receipt, (2) receiving price-spike vs weighted-average cost, (3) theoretical-vs-actual usage variance (recipe explosion × sales vs stock issues), (4) sales anomaly vs 12-month average, (5) margin/loss-making items below GP norm. Runs **reactively** (screens every stock-in receipt before approval — wired into `stock-in.actions.ts`) and **proactively** (`runPeriodicScan`). Findings carry severity, human-readable detail, and estimated financial exposure (cents), are **deduped** and persisted to `fte_alerts`, surfaced in a dashboard feed (`FtePanel` on `/dashboard`), and **pushed out-of-band via branded HTML email (nodemailer) + WhatsApp bridge** (`notify.ts`) — tagline in the email: *"Detected automatically… no one had to open a report."* Toggle stored on operational settings. Verified by `src/db/test-fte.ts` (e.g. a 10,000-case-vs-10,000-pcs receipt correctly flags wrong_unit) and `run-fte-scan.ts` against real data.

**SOP knowledge base:** `recipe_sops` / `recipeSopLinks` tables populated by Python extractors (`scripts/extract_sop*.py`) with dish summary, components, method, cook params, allergens, critical points, packaging + images (`public/sop-images/`), viewable at `/menu/recipes/[id]/sop` and injected into the advisor.

## 3. Planned but missing
- **⛔ Payroll — explicitly PARKED** (Inngest jobs; the entire module is intentionally unbuilt; nav links removed to avoid 404s). Spec exists at `spec/payroll.md`.
- **No real scheduler.** The proactive Auto-FTE scan is only a manual one-off script (`run-fte-scan.ts`); there is no cron/Inngest/queue actually wiring periodic execution. "24/7" is not yet literally true.
- Deferred-by-design tail (per Phase-3 waves): multi-location salary distribution, per-location stock reorder alerts, HR filters modal, reward-points engine, deal dynamic-pricing, time-windowed promos, Excel/PDF/thermal exports (only CSV), RBAC key-name reconcile, binary logo/file upload (URL only).
- Inngest, Dinero.js, and Temporal were named in the build plan but **not installed** (money is hand-rolled integer cents; no Inngest in `package.json`).
- Real `.eml/.msg` email-thread parsing for change-control (files stored verbatim).
- Out of scope entirely: the operational POS / Call-Center front-ends, App Store, Floor Plan.

## 4. The AI gap
- **The AI is real but narrow and bolted-on.** Of 113 routes, ~110 are classic deterministic CRUD; AI lives in exactly two slices (recipe advisory + inventory/sales anomaly monitoring). The product's identity is still "legacy portal clone," not "AI-native platform."
- **Advisory, not actuating.** Both AI features stop at recommendation/alerting. The Recipe Advisor cannot itself open a Recipe Change Request; the Auto-FTE agent flags exposure but cannot hold/reject a suspicious receipt, re-price a loss-making item, or raise a PO — a human still acts. No closed loop.
- **Not truly autonomous yet.** No scheduler means the "proactive" agent only runs when a human invokes a script. No background runtime, no per-tenant scheduling, no retries/queue.
- **Provider posture is off-brand for an Anthropic-native shop:** primary is OpenAI with Gemini fallback; Claude is referenced only in a code comment. Models list includes speculative ids (`gpt-5`, `gemini-3-pro-preview`).
- **Governance/observability light:** no token/cost accounting, no eval harness for advisor answer quality, no guardrails on AI-suggested quantity/price changes, no audit trail tying an AI suggestion to the change it influenced. Notifications are demo-grade (single hard-coded recipient via env; WhatsApp via a local bridge).
- **Single-tenant reality:** every table carries `tenant_id`, but the AI features are exercised only against the one Nandos tenant with hand-seeded sales/costing data.

## 5. Missing pieces to make it sellable
- **Wire a real scheduler** (Inngest/cron/queue) so `runPeriodicScan` runs per-tenant on a cadence — this is the single change that makes "24/7 autonomous FTE" literally true.
- **Close the loop:** let the Auto-FTE agent take guarded actions (auto-hold a flagged receipt pending approval, draft a re-price proposal, open a Recipe Change Request from an advisor recommendation) behind human-approval guardrails.
- **Multi-channel hardening:** move off the single env-recipient + local WhatsApp bridge to a real per-tenant notification config, delivery retries, and an in-app inbox with ack/resolve workflow (schema already has `status: open|ack|resolved`).
- **Commit and document the AI work** (it is uncommitted and absent from CHECKPOINT.md) and add a hosted demo with seeded anomalies.
- **Trust layer:** token/cost metering, an answer-quality eval set, citation of which SOP/costing row drove each suggestion, and an audit link from AI suggestion → applied change.
- **Tenant onboarding:** an ETL/self-serve path to ingest a new chain's menu/costing/sales so the agents have data to reason over.
- Standardize the model catalog on supported, dated model ids; add a provider abstraction that includes Claude.

## 6. Native-AI + Autonomous-FTE upgrade
- **AI Food-Cost Controller (FTE) — the flagship persona.** Extend the existing Auto-FTE agent into a true 24/7 employee: continuously watches receiving, stock issues, recipe variance, sales and margins; auto-holds suspicious receipts; drafts price corrections and supplier-dispute notes; escalates by WhatsApp/email with quantified exposure. *Replaces ~the bulk of a junior cost-controller / inventory-audit role (~120–160 hrs/month of manual report-pulling and reconciliation).* Guardrails: read-only by default, all actuating actions require human approval, hard caps on auto-actions, full audit trail, dedupe to prevent alert spam.
- **AI Recipe/Menu Engineer (FTE).** Promote the Recipe Advisor from chat to a standing agent that proposes portion/ingredient/price changes as approval-ready Recipe Change Requests, grounded in SOP + costing + sales volume. *Replaces ~40–60 hrs/month of chef + finance recipe-engineering work.* Guardrails: segregation-of-duties already enforced by the change-control module; AI can only propose, never self-approve.
- **AI Inventory/Replenishment agent.** Drive demand notes → POs from theoretical usage + reorder levels + lead times. *Replaces ~60–80 hrs/month of manual ordering.* Guardrails: spend ceilings, approved-supplier lists, human sign-off above a threshold.
- **AI Ops Manager (cross-module).** A daily digest agent that reads the alert feed + KPIs and produces a prioritized action list per location. Guardrails: summarize-and-recommend only.

## 7. Showcase angle (for the portal)
- **Headline:** "OperaNova Auto-FTE — the back-office employee who never opens a report, because it already read them all."
- **Stat-benefit bullets:**
  - **5 live anomaly detectors** (wrong-unit receipts, price spikes, usage variance, sales dips, thin margins) each flagged with **Rupee exposure attached** — catches the costly mistakes humans miss at the receiving door.
  - **Reactive + proactive in one agent:** screens every delivery *before* it's approved and sweeps usage/sales/margins on a schedule — then pushes findings to **WhatsApp and email automatically**, no dashboard visit required.
  - **Grounded, not hand-wavy:** advice and alerts are built from the chain's **real recipes, SOPs, costing workbook and 12-month sales**, with a chef-bot and a finance-bot that **consult each other** before answering.
- **Demo hook:** Receive 10,000 *cases* of a sachet (a fat-finger for 10,000 *pieces*) → the agent instantly flags a critical "12+ months of stock in one delivery" over-receipt with the implied loss in cents, fires a branded WhatsApp + email, and drops it on the dashboard feed — live, in under a second.

## 8. Verdict
- **AI-native score:** 3/5 — two genuinely agentic, well-grounded features (tool-calling cross-consultation; reactive+proactive multi-channel monitoring) sit on a large, deterministic CRUD core; AI is impactful but confined and not yet scheduled or actuating.
- **FTE-fit score:** 4/5 — the "Auto-FTE" agent is almost a literal embodiment of the autonomous-employee thesis (named as such, replaces a real role, alerts out-of-band with quantified impact); held back only by the missing scheduler and advisory-only (non-actuating) posture.
- **Maturity:** MVP (strong) — production-grade clone core; AI features working and integration-test-verified but uncommitted, single-tenant, demo-grade notifications, no hosted demo.
- **Recommendation:** **Secondary** — showcase now via the Auto-FTE narrative (it is the most on-thesis asset in the portfolio), with a clear, short path to **Flagship**: wire a real scheduler, close the action loop with guardrails, and ship a hosted multi-tenant demo.
