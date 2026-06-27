# Personal AI Employee (Digital FTE) — A local-first, human-in-the-loop "autonomous employee" that triages email/WhatsApp, answers customers, and builds finance reports 24/7

## 1. Snapshot
- **One-liner:** An Obsidian-vault-backed "AI Employee" that watches Gmail/WhatsApp/filesystem, classifies and routes work through a Kanban (Inbox → Needs Action → Pending Approval → Done), auto-replies to common requests, generates LLM-driven Excel sales reports, and pulls live Xero financials into CEO briefings — with a cloud worker drafting replies while you sleep.
- **Category:** Autonomous back-office / personal-business operations agent (customer-service + finance-ops automation).
- **Target buyer:** SMB owner-operators and single-brand operators (built around Nando's Pakistan / "Anjum AI Solutions"); the hackathon framing targets anyone wanting to "hire" a Digital FTE instead of headcount.
- **Tech stack:** Python 3.13, FastAPI + WebSocket backend; React 18 + Vite + TypeScript frontend; Google Gemini (`google-genai`, multi-key rotation, `gemini-1.5-flash`) as primary LLM with OpenAI `gpt-4o-mini` fallback; PostgreSQL (sales data) via raw SQL + Alembic; Xero OAuth2 (accounting MCP); Gmail OAuth (real send/receive); WhatsApp via the `whatsapp-mcp` Go bridge SQLite store; `watchdog` filesystem watchers; openpyxl for Excel; vault = Obsidian markdown + Git sync; cloud worker on an Oracle Cloud VM under PM2.
- **Maturity:** **MVP** (working, demoable, single-tenant). Real OAuth integrations, real email sending, real LLM report generation, a live React dashboard and a deployed cloud worker — but hardcoded to one user/brand, no auth, no multi-tenancy, and the headline "reasoning" flow is largely a rule engine (see §4).
- **Live URL / demo:** Local only — backend `http://localhost:8000`, frontend `http://localhost:5173`. Cloud worker VM referenced at `161.118.232.249` (Oracle Cloud); GitHub `ShehzadAnjum/Hackaton_0_FTE`. No public hosted URL.
- **Path:** /home/anjum/dev/Hackaton_0_FTE

## 2. What's built today (verified)
- **Watcher-driven ingestion pipeline (real).** A background thread polls Gmail every ~30s, WhatsApp every ~10s, and watches the Inbox folder, auto-moving items through vault folders. Evidence: `src/api/main.py` (`run_inbox_watcher_thread`), `src/watchers/{gmail_watcher,whatsapp_watcher,filesystem_watcher}.py`.
- **Gmail send + receive (real OAuth).** Reads unread mail and sends formatted HTML/plain replies with PDF attachments. Evidence: `src/services/gmail_oauth_service.py`, `src/utils/email_sender.py`, `config/token.json`/`credentials.json` on disk.
- **WhatsApp ingestion (real, via whatsapp-mcp).** Reads the Go bridge's SQLite DB (`~/whatsapp-mcp/whatsapp-bridge/store/messages.db`) into the vault. Evidence: `src/services/whatsapp_reader.py` (uses `aiosqlite`), live messages in `vault/_Live_WhatsApp/`.
- **Rule-based classification + routing engine (real, deterministic).** ~800-line keyword/regex/threshold classifier that routes to auto-approve / approval-required / escalation / rejected, with menu/HR/report/invoice/Xero/financial-threshold logic. Evidence: `src/api/processor.py` (`RuleProcessor.classify_item`).
- **Human-in-the-loop approval workflow (real).** Pending_Approval → Approve/Reject → Done/Rejected, with `Plan.md` files and audit logging. Evidence: `vault/Pending_Approval/*.md`, approve/reject endpoints in `src/api/main.py`, `src/services/approval_workflow.py`.
- **LLM-powered Excel report generator (genuinely AI).** Gemini converts a natural-language email request + DB schema into JSON report specs (SQL + chart type), then builds a multi-sheet, charted, totaled Excel workbook; has a rule-based fallback if the LLM fails. Wired into the approval flow. Evidence: `src/services/llm_report_generator.py`, `src/services/sales_llm_query.py`, `src/services/db_metadata.py`; called from `src/api/processor.py:1457` and `src/services/approval_workflow.py`; ~30 generated `.xlsx` files in `reports/`.
- **Xero accounting integration (real OAuth2).** Org details, invoices, contacts, P&L, chart of accounts, and create-invoice. Evidence: `src/mcp/xero_config.py`, `src/services/xero_service.py`, `/api/xero/*` endpoints, `config/xero_tokens.json`, `test_xero.py`.
- **Auto-responder for pre-approved requests (real).** Explicit menu requests → emails `menu-pk.pdf`; HR-policy requests → emails `HumanResource.pdf`; with duplicate-send suppression. Evidence: `RuleProcessor._send_menu_email` / `_send_hr_policy_email`.
- **CEO Daily Briefing / Weekly Audit (real data, template-generated).** Pulls live Xero figures + vault/audit stats into a markdown briefing emailed as styled HTML. Evidence: `scripts/daily_briefing.py`, `scripts/weekly_audit.py`, `/api/gold/*`, `vault/Briefings/*` (20+ dated briefings).
- **Production-grade Gemini reliability layer (real).** Multi-key pool rotation (up to 20 keys), quota/429 handling, exponential backoff, JSON repair. Evidence: `src/services/gemini_service.py`.
- **Cloud/Local "Platinum" split (real, modest).** A separate `cloud_worker/` service polls Gmail, generates Gemini draft replies into `Pending_Approval/cloud/`, and Git-syncs the vault; local side approves and executes the send. Evidence: `cloud_worker/{main,draft_generator,gmail_monitor,vault_sync}.py`, `vault/Updates/cloud_worker_status.json`, `/api/platinum/*`.
- **React dashboard (real).** Kanban board, sources panel, task detail sidebar, Gold-tier panel, LLM log viewer, oversight stats, WebSocket live updates. Evidence: `frontend/src/components/*`, `frontend/src/App.tsx`.
- **Audit logging + LLM call logging (real).** JSON audit trail and per-request LLM logs. Evidence: `src/services/audit_logger.py`, `src/services/llm_logger.py`, `vault/Logs/`, `logs/llm/`.
- **Claude Code skill/hook scaffolding (present).** `.claude/skills/*` (ceo-briefing, email-classification, invoice-creation, xero-accounting, facebook-instagram, twitter-x), `skills/nandos-report-generator`, and a Ralph-Wiggum stop hook `.claude/hooks/ralph-wiggum.sh`.

## 3. Planned but missing
- **Social media actions are scaffold-only.** Facebook/Instagram and Twitter/X exist as `.claude/skills/*/SKILL.md` definitions and DEMO_GUIDE talking points, but there is no working posting integration in the backend (no FB/Twitter service or API route).
- **Banking integration is a stub.** `Dashboard.md` shows "Primary Account: -- / Not connected" and a "Connect banking integration" note; the "audits bank transactions" CEO-briefing vision is fulfilled via Xero, not a bank feed.
- **"Claude reasoning loop."** The hackathon doc and DEMO_GUIDE describe Claude Code as the reasoning brain with Ralph-Wiggum iteration; the running backend uses a Python rule engine. The `.claude/skills` and hook are present but not what executes in the live demo (see §4).
- **OpenAI fallback for classification** is coded but only active if `OPENAI_API_KEY` is set; primary path is Gemini.
- **Multi-restaurant data.** Schema/UI imply multiple outlets, but the seeded sales DB is effectively one restaurant ("Boat Basin", Karachi) per `sales_llm_query.py`.
- **No tests of substance running.** `tests/` (unit/contract/integration) directories exist but coverage/CI is unverified; correctness rests on demo scenarios in `vault/_Test_Scenarios/`.

## 4. The AI gap
- **The flagship demo flow is rule-based, not LLM.** The live dashboard pipeline routes everything through `RuleProcessor` (`src/api/main.py` → `processor.process_item`), which is pure keyword/regex/threshold logic. A separate LLM classifier exists (`src/services/classifier.py` → `src/agent/tools/classify_tools.py` calling Gemini), but it is **not** what the running backend uses to move Kanban cards. Two parallel classifiers exist; the demo shows the deterministic one.
- **`Plan.md` "reasoning" is a template, not Claude.** `RuleProcessor._create_plan_file` fills a fixed checklist from rule outputs — labeled "Claude reasoning loop / Plan.md" in DEMO_GUIDE, but no LLM produces it.
- **CEO Briefing "Proactive Suggestions" are threshold rules.** `scripts/daily_briefing.py` builds suggestions from `if` thresholds (pending count, draft-invoice count, outstanding amount, error count) — real Xero data, but no LLM analysis or narrative reasoning.
- **Where the AI is genuinely real:** (1) **NL→SQL→Excel report generation** via Gemini (`llm_report_generator.py`, `sales_llm_query.py`) — the strongest, non-trivial AI feature, with graceful rule-based fallback; (2) **cloud-worker draft replies** via Gemini (`cloud_worker/draft_generator.py`), though it keyword-classifies first and uses canned fallbacks; (3) the Gemini reliability/rotation layer. Net: real LLM surface exists but is narrower than the "autonomous AI employee" marketing implies.
- **Honest characterization:** ~70% deterministic rules + integrations, ~30% real LLM. The product is a solid integration/automation harness with selective AI, not an LLM-reasoning agent end-to-end.

## 5. Missing pieces to make it sellable
- **Auth & multi-tenancy:** no login, no user/org model; single-user values are hardcoded (`sanjum77@gmail.com`, `nandospak.com`, company contact lists, VM IP `161.118.232.249`). Needs tenant isolation for vault, DB, and OAuth tokens.
- **Secrets handling:** live `.env`, `config/token.json`, `config/credentials.json`, `config/xero_tokens.json` sit on disk (gitignored, but a `client_secret_*.json` artifact is present). Needs a secrets manager and per-tenant credential vaulting before any hosted offering.
- **Billing & packaging:** no subscription/metering; the "Digital FTE pricing" ($500–$2,000/mo) is narrative only. Needs usage metering (LLM tokens, emails, reports) and plans.
- **Onboarding:** setup is a 4-terminal manual process (`README.md`/`notes.md`) plus OAuth dances and a Go WhatsApp bridge. Needs guided connect-your-accounts onboarding and one-click deploy.
- **Reliability/ops:** single Oracle VM + PM2, Git-as-message-bus, in-memory dedupe sets. Needs a durable queue, health/alerting, and idempotency guarantees.
- **Compliance:** customer email/WhatsApp PII flows through LLMs; needs data-retention, consent, and audit-export controls (audit logging exists as a foundation).
- **Demo data & generalization:** decouple from Nando's-specific menu/HR PDFs and the single sales schema; provide seed data and a config-driven "company handbook."
- **Consolidate the two classifiers** and make the LLM path the default so the product behaves as advertised.

## 6. Native-AI + Autonomous-FTE upgrade
This is already framed as a Digital FTE; the upgrade is to make the AI do the reasoning the marketing claims, and package it as named "employees":
- **Customer-Service FTE (front desk):** Replace the menu/HR/hours keyword rules with an LLM intent+RAG responder over a per-tenant knowledge base; auto-handle Tier-1 inquiries, escalate the rest. *Replaces ~20–30 hrs/week of a support rep.* Guardrail: confidence threshold + human approval for anything financial/legal/VIP (already modeled in `RuleProcessor`).
- **Finance-Analyst FTE (the genuine differentiator):** Productize the NL→SQL→Excel generator + Xero briefings into a "ask your business a question, get a charted report" agent. *Replaces ~10–15 hrs/week of analyst/bookkeeping reporting.* Guardrail: read-only SQL sandbox, approval before any Xero write (invoice create already gated).
- **Email-Triage / EA FTE (24/7):** Upgrade the cloud worker from keyword-classify + canned drafts to LLM triage with summaries, suggested actions, and draft replies queued for one-tap approval. *Replaces ~10 hrs/week of inbox management.* Guardrail: cloud drafts only; local human approves the send (work-zone split already built).
- **Reframe the headline metric** around the doc's own "168 hrs/week, ~$0.50/task vs ~$5.00" story, but label projected savings as targets, not measured.
- **Human-in-the-loop is the moat, not a limitation:** the vault-as-audit-trail + approval folders are a clean, demoable governance story for risk-averse SMB buyers.

## 7. Showcase angle (for the portal)
- **Headline:** "Hire a Digital FTE: a 24/7 AI employee that triages your inbox, answers customers, and writes your finance reports — you just approve."
- **Benefit bullets:**
  - Works **168 hrs/week vs a human's ~40** at a fraction of the cost — *aspirational target, from the hackathon economic model, not measured in-product.*
  - **Real integrations, today:** Gmail (send/receive), Xero (live P&L + invoices), WhatsApp, and a React oversight dashboard — not a mockup.
  - **Every action is logged and human-approved** — a built-in audit trail and approval queue (governance-first).
- **Demo hook:** Email "send me the Boat Basin sales report, item-wise and channel-wise with charts" → the agent uses Gemini to turn that sentence into SQL, runs it against the Postgres sales DB, and emails back a multi-sheet, charted Excel workbook in seconds. (This is the one flow where the AI genuinely reasons — lead with it.)

## 8. Verdict
- **AI-native score:** 3/5 — real, non-trivial Gemini features (NL→SQL reporting, draft generation, robust key-rotation layer), but the marquee dashboard flow and "reasoning"/"briefing intelligence" are deterministic rules dressed as AI.
- **FTE-fit score:** 4/5 — purpose-built for the autonomous-FTE thesis: watchers wake it, it works 24/7 via a cloud worker, human-in-the-loop approval, vault-as-memory. Best thematic fit in the portfolio.
- **Maturity:** MVP (single-tenant, demoable, real OAuth + real sends; no auth/billing/multi-tenancy).
- **Recommendation:** **Flagship** — it is the strongest embodiment of the "Restless Workforce" story and the only project with a believable end-to-end FTE narrative; ship it as the hero demo, but lead with the genuine LLM report flow and harden the AI claims (consolidate to the LLM classifier, drop "Claude reasoning" language where it's actually rules) before any customer-facing pitch.
