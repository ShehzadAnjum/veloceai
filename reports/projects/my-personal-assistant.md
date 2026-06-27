# My Personal Assistant (MPA) — A 24/7 AI inbox-and-delegation chief-of-staff for Gmail + WhatsApp

## 1. Snapshot
- **One-liner:** Always-on assistant that watches two Gmail accounts and WhatsApp, classifies every message with an LLM, drafts/sends replies in the owner's voice, and auto-creates and chases delegated tasks.
- **Category:** AI communications / inbox-zero + delegation automation (personal-productivity / ops back-office).
- **Target buyer:** Today it is built for a single power user — an IT manager at Nando's Pakistan juggling personal + work Gmail, a WhatsApp owner number, and a team he delegates restaurant ("CASA") IT issues to. Generalizes to any busy operator/exec who lives in email + WhatsApp.
- **Tech stack:** Backend — Python 3.11+, FastAPI, SQLModel/SQLAlchemy 2.0, Alembic, PostgreSQL 16 (asyncpg); LLM — Google Gemini 2.5 Flash (primary, 10-key rotation) with OpenAI `gpt-4.1-mini` fallback; integrations — Gmail API (OAuth2), Google Calendar API, a Go WhatsApp bridge (whatsmeow) read via SQLite + sent via local REST. Frontend — React 18 + TypeScript 5 + Vite 5 + Tailwind v4 + shadcn/ui + TanStack Query. Deploy — Oracle Cloud VM, nginx, systemd units + a bridge watchdog timer.
- **Maturity:** **MVP (deployed, single-tenant).** Justification: this is not a prototype — it is a working, deployed system. It has real OAuth into two live Gmail accounts, a paired WhatsApp bridge, six distinct LLM-powered services, 15+ DB tables with 13 Alembic migrations, ~114 backend pytest functions across 11 files, a full multi-page React UI wired to the real API (not mocks; `VITE_USE_MOCK_DATA` defaults off), and production deploy scripts (nginx + systemd + watchdog) targeting a public VM. It falls short of "Production" because it is single-user with **no authentication/multi-tenancy**, autonomous sending is gated behind `testing_mode`/`email_testing_mode = True` by default, and live API keys + OAuth client secrets are committed into the repo.
- **Live URL / demo:** `http://158.178.227.174` (Oracle Cloud VM per `deploy/.env.production` + `deploy/nginx.conf`). No public/authenticated demo; single-owner instance.
- **Path:** /home/anjum/dev/my_personal_assistant

## 2. What's built today (verified)
Verified by reading the source, not the planning docs. (Note: `specs/001-unified-comms-hub/gap-analysis.md` is a **stale snapshot** — it lists many items as "missing," but the git log shows "Tier 2–5" closure commits and the code confirms they shipped.)

**Channels — both real:**
- **Gmail (2 accounts: personal + official):** `EmailWatcher` polls every 120s, does a smart "deep sync" (only fetches after the latest stored message), runs API calls in a thread pool. `gmail_service` has working `fetch_messages` and `send_email` (MIME via Gmail API). (`services/watchers.py`, `services/gmail_service.py`)
- **WhatsApp:** reads the Go bridge's SQLite store (`messages.db` + whatsmeow `whatsapp.db`) with sophisticated JID/LID→phone→contact name resolution; sends via the bridge's local REST API. `WhatsAppWatcher` polls every 10s. QR pairing UI exists. (`services/whatsapp_service.py`, `routes/whatsapp.py`)

**AI classification — real and central:** Every incoming email is classified by Gemini into intent, `urgency_score` 0–100 (deterministically mapped to a Priority enum), entities, summary, proposed due date, plus domain-specific fields (delegation_detected/target, complaint CASA/region, delegate response_status). Few-shot, JSON-structured, with safe-default fallback on parse failure. (`services/classifier.py`, `services/email_processor.py`)

**Auto-reply — built, two paths:**
- *WhatsApp:* rule engine (regex/keyword, DB-backed, 60s cache) matches inbound messages and sends a templated reply; supports owner commands over WhatsApp ("auto reply on/off", "switch to live/testing"); de-dupes already-answered threads. (`services/auto_reply_processor.py`, `services/rule_engine.py`)
- *Email:* rule match → either fixed-text or **LLM-generated** draft. `reply_generator.py` composes the reply using the contact's style profile + few-shot sample replies (with PRA↔FBR synonym expansion). Drafts default to pending approval; **auto-send fires only when a template is `AUTO_SEND` and the contact is `TRUSTED`**, in which case `gmail_service.send_email` is called and an audit log + WhatsApp confirmation are written. (`services/email_processor.py`)
- *Approval workflow:* the `/approvals` PATCH route genuinely sends the email on approval (sets status `SENT`), and edited approvals feed the learning loop. (`routes/drafts.py:217-292`)

**Delegation pipeline — built (this is the standout):** classifier detects delegation → `_handle_delegation` auto-creates a `Task` assigned to the matched contact with rich `delegation_metadata`; `_check_completion_signal` watches replies for "done/resolved/fixed" and auto-closes the matching task; `reminder_processor` runs 24/7 sending due reminders and **auto-chasing overdue delegations** via WhatsApp to the owner + a "gentle reminder" email to the delegatee; a batch scanner back-fills historical delegation threads. (`services/email_processor.py`, `services/reminder_processor.py`, `services/batch_scanner.py`)

**Task management:** full CRUD + delegation views ("Arshad overview", delegation stats), scheduled reminders (WhatsApp/email channels), batch scan endpoint. (`routes/tasks.py`)

**Learning loop — built:** when a user edits a draft, `feedback_learner.py` uses the LLM to classify the edit, store a suggestion, and auto-create a `ReplyStyleSample` + update the per-contact `ReplyStyleProfile` so future drafts improve. `rule_enhancer.py` uses the LLM to expand a one-word trigger into comprehensive regex (incl. Roman-Urdu transliterations).

**Study Hub (feature 002):** LLM extraction of a prioritized learning backlog (paths/assignments/tasks/resources) + per-group and cross-group digests from WhatsApp AI-course groups (mentor "Zia Khan"); idempotent upserts, batched to respect Gemini quota. (`services/study_hub_service.py`, `routes/study.py`)

**Real-time + supporting:** WebSocket broadcasts (`ws_manager`) for new messages/drafts/stats; Google Calendar service (real API, degrades gracefully if no token); contact groups model exists; CASA/CMS import; audit log; notifications log; behavior-learning dashboard. Frontend covers Inbox, Approvals, Contacts, Rules, Templates, Tasks, Delegations, Calendar, Study Hub, Chat, Audit, Learning, Settings, Control pages.

## 3. Planned but missing
- **Truly agentic chat.** `routes/chat.py` advertises action-taking and even defines a `tool_calls` schema, but the implementation only stuffs DB context into a prompt and returns text — `tool_calls` is **always `None`**. The spec's "AI Instruction Chat Box" (US9: type a command, it executes multiple actions) is **not** shipped; what exists is read-only Q&A over the inbox.
- **Study Hub Phase 2 (explicitly out of scope in the spec):** auto-reminders/calendar sync for learning items, private 1:1 messages, non-Zia authors, unrelated chats. Item status tracking is only partially wired.
- **Multi-tenancy / auth:** no login, no user model — the whole app assumes one owner. The owner phone (`923332301127`), Nando's-Pakistan domain prompts, CASA list, and the delegate "Arshad" are hardcoded.
- **Autonomy is off by default:** WhatsApp auto-reply ships in `testing_mode=True` (diagnostics to owner, not the sender) and email runs in `email_testing_mode=True`; full autonomous send requires per-contact trust + per-template opt-in.
- **Retrieval is keyword/ILIKE, not semantic** — reply-style sample matching uses subject keyword + hardcoded synonyms, not embeddings.

## 4. The AI gap
What's genuinely AI-native vs. aspirational:
- **Real LLM, deeply woven (not bolt-on):** six distinct production LLM workloads — classification, reply generation, feedback-edit analysis, trigger/rule enhancement, study extraction, and inbox Q&A — on a resilient provider layer (10-key Gemini rotation, quota/auth/transient error handling, OpenAI fallback). This is the strongest part of the product.
- **But it is not an "agent."** Actual actions (send/auto-reply) are driven by **deterministic regex/keyword rules** and human-approval gates, not by an LLM planner with tool-use. There is no plan→act→observe loop, no function-calling, no autonomous decision to send without a pre-trusted rule. The classifier is single-shot; there is no self-correction or escalation reasoning.
- **Provider note:** this is a **Gemini/OpenAI** system. There is no Anthropic/Claude usage anywhere in the codebase.
- **The gap to "autonomous FTE":** the intelligence (read, understand, draft, prioritize, detect delegation) is there; the **autonomy and agency** (decide and act end-to-end, safely, without per-rule pre-authorization) is the missing layer.

## 5. Missing pieces to make it sellable
- **Security hygiene (blocker):** live Gemini API keys and Gmail/Calendar OAuth client secrets + tokens are committed to the repo (`deploy/.env.production`, `backend/credentials/*.json`). Must be rotated and moved to a secrets manager before any showcase.
- **Multi-tenant + auth:** user accounts, per-tenant credential vaults, and de-hardcoding the owner/CASA/Arshad/Zia specifics into configurable workspaces.
- **Productize autonomy with guardrails:** confidence thresholds, allow/deny lists, "auto-send under $X / below urgency Y" policies, and an audit/undo trail (the audit log is a good start) so a buyer can dial autonomy from "suggest" → "send."
- **Agentic chat for real:** wire the existing `tool_calls` schema to the service layer (create task, send reply, set priority, schedule reminder) so the chat box actually executes.
- **Reliability/observability:** the WhatsApp bridge is the fragile dependency (a watchdog exists); add health dashboards, ret[r]y/dead-letter on sends, and per-tenant quota metering.
- **Onboarding:** self-serve OAuth + WhatsApp pairing flow, and a setup wizard instead of hand-edited `.env`.

## 6. Native-AI + Autonomous-FTE upgrade
Position MPA as **"Aria — your AI Chief of Staff for the inbox,"** an autonomous FTE that works a 24/7 shift across email + WhatsApp:
- **Inbox-zero agent:** reads, classifies, and triages every message; drafts on-brand replies in the principal's voice; auto-sends the routine tier under policy; surfaces only the exceptions. Replaces **~15–25 hrs/week** of an EA/inbox-manager's triage + drafting.
- **Delegation chief-of-staff:** detects "please handle this," opens and assigns a tracked task, pings the owner, chases the delegatee on overdue, and auto-closes on a "done" reply — replacing the manual follow-up grind of an ops coordinator (**~5–10 hrs/week**).
- **Upgrade path to true agency:** add an LLM planner with tool-use over the (already-built) action surface (send email, create/assign task, schedule reminder, set priority), plus a memory/profile store (already exists as ReplyStyleProfile) for voice consistency.
- **Guardrails (table-stakes for the FTE pitch):** tunable autonomy levels (suggest / approve / auto), per-contact trust, urgency/confidence thresholds, allow-lists, full audit log + one-click undo, and the existing "testing mode" as the safe default. This is what lets a buyer trust an agent with their outbound.

## 7. Showcase angle (for the portal)
- **Headline:** "The teammate who never sleeps on your inbox — triages every email and WhatsApp, replies in your voice, and chases your delegations until they're done."
- 3 stat-benefit bullets:
  - **2 inboxes + WhatsApp, watched 24/7** — every message LLM-classified for urgency in seconds; urgent items hit your phone within ~30s.
  - **Replies in your voice, not a template** — Gemini drafts using your per-contact style profile and learns from every edit; trusted routine mail can auto-send.
  - **Delegations that close themselves** — it spots a hand-off, opens and assigns the task, nags the overdue ones by email + WhatsApp, and marks it done when the reply says "done."
- **Demo hook:** Send a "POS is down at Sehar, please fix ASAP" email to the watched inbox. Live on screen: it's classified TOP_URGENT, a delegation task is auto-created and assigned, the owner gets a WhatsApp alert in seconds — then reply "fixed, replaced the router" and watch the task auto-close.

## 8. Verdict
- **AI-native score:** 4/5 — six real, well-engineered LLM workloads woven through the core; loses a point because the *actions* are rule-driven, not agentic (no tool-use/planner), and retrieval is keyword-based.
- **FTE-fit score:** 4/5 — among the strongest "replaces a role" narratives available: a concrete inbox+delegation EA/chief-of-staff that already runs 24/7. Held back from 5 by gated autonomy (testing-mode default, per-rule pre-authorization) and the lack of true end-to-end agency.
- **Maturity:** MVP (deployed, single-tenant; ~114 backend tests; live on an Oracle VM).
- **Recommendation:** **Secondary — on the flagship track.** It is the most on-narrative, most complete "autonomous FTE" asset in the portfolio and demos beautifully. Promote to **Flagship** once three things land: (1) rotate/secure the committed secrets, (2) multi-tenant + de-hardcode the single-owner specifics, and (3) ship policy-gated autonomous send + a genuinely agentic chat. Until then, showcase it as the lead "autonomous inbox/delegation FTE" story with a scripted live demo.
