# Hamdan ERP — Multi-tenant SaaS accounting/ERP kernel for holding companies (Pakistan-first)

## 1. Snapshot
- **One-liner:** A double-entry, multi-tenant SaaS ERP whose first customer (Al-Hamdan Enterprises) runs a perfume-retail business on it; the accounting + inventory + sales/purchase core works end-to-end in the browser.
- **Category:** B2B SaaS — accounting / ERP platform (mid-market, holding-company / multi-entity focus).
- **Target buyer:** SME holding groups in Pakistan with heterogeneous sub-businesses (retail, services, education) that need consolidated, compliant books; first concrete buyer is a perfume retailer migrating off MS Access.
- **Tech stack:** Next.js 15 + Tailwind v4 + shadcn/ui + Better Auth (web/BFF); FastAPI + SQLModel + Alembic + asyncpg + structlog (API); PostgreSQL 16 with forced Row-Level Security; Docker Compose local; planned Vercel + Fly.io (Mumbai) + Neon (Singapore) + Cloudflare R2.
- **Maturity:** **MVP.** Justification: 15 Alembic migrations, 71 tables / 10 views / 26 triggers, RLS enforced via a de-privileged `hamdan_app` role, a real weighted-average-cost inventory engine, and all four money cycles (sell / receive / buy / pay) posting balanced journal entries through the browser. A documented testing pass (#1, 2026-05-26) ran 68/79 checkpoints with zero failures and a trial balance that balances to the rupee. It is not Production: not deployed to cloud, no billing, only one of three planned companies' modules built, tax engine deferred.
- **Live URL / demo:** None. Local only (`localhost:3000` web, `:8000` API docs, `:8080` Adminer). Cloud deploy is queued (Phase 1.5). Public-domain rule documented but unrealized.
- **Path:** /home/anjum/dev/hamdan_ent

## 2. What's built today (verified)
Evidence is file/dir-level in the repo.

- **Three-level tenancy schema + real RLS isolation.** `org_id` / `company_id` / `branch_id` model; RLS forced on per-tenant tables (migration `0003_enable_rls.py`). `apps/api/src/hamdan_api/db.py` opens each request transaction with `SET LOCAL ROLE hamdan_app` (drops BYPASSRLS) + `set_config('app.org_id', …)`. Testing pass confirms a stranger user in another org sees zero rows on every protected endpoint.
- **Session auth across the web/API boundary.** `apps/api/src/hamdan_api/auth.py` forwards the Better Auth cookie to `/api/auth/get-session`; `require_session` / `require_org` guards. Better Auth `organization` plugin tables exist (migration `0002`).
- **Hierarchical IFRS Chart of Accounts.** 5 seeded account classes → per-tenant subclasses → accounts (parent_id + level, postable leaves). Migration `0005`, UI `(app)/accounts/`.
- **Immutable double-entry journal.** Migration `0009_journal_and_balance_trigger.py` + 5 triggers: balance (Dr=Cr), posted-row immutability whitelist, no-delete-posted, line-protect, and auto-emit `journal_entry.posted/.voided` into `event_log`. Service `services/journal_posting.py` (N-line balanced JV).
- **Documents-create-postings model, fully wired for 4 flows:**
  - Sales invoice → stock issue + balanced JV with COGS at WAC (`services/sales_posting.py`, `(app)/sales/invoices/`).
  - Purchase invoice → stock receipt refreshing WAC + JV Dr Inventory/Cr AP (`services/purchase_posting.py`).
  - Customer receipts with split allocation across open invoices (`services/customer_payment_posting.py`).
  - Vendor payments with split allocation (`services/vendor_payment_posting.py`).
- **Weighted-average-cost inventory engine.** Migration `0010` — append-only `stock_valuation_layers`, trigger refreshes WAC on receipt, stamps issue lines, blocks negative stock. UI: products list/CRUD, stock-on-hand, movements.
- **Master-data CRUD through the UI.** Products, customers (parties + `customer_profiles`), vendors (parties + `vendor_profiles`) — two-table writes. `(app)/customers/`, `(app)/vendors/`, `(app)/inventory/products/`.
- **Banking + tax + integration-kernel schema.** Banking (migration `0011`: bank_accounts, statements, reconciliations), tax framework (`0007`: authorities/codes/rules/registrations, seeded FBR + 2 codes), integration kernel (`0008`: `event_log`, `external_id_mappings`, `api_keys`, `webhook_subscriptions`).
- **Reporting views.** Migration `0014` — 10 `security_invoker` views: trial balance, balance sheet, income statement, AR/AP aging, party balances, stock-on-hand, journal-with-details, etc. Dashboard KPIs read these (`routers/dashboard.py`, `(app)/dashboard/`).
- **Seeds that boot a working system.** Universal seed (8 currencies, 63 countries, 59 UoMs) + a Python Uroojj demo seed (75 bilingual accounts, 5 products, 2 customers, 3 vendors, 2 banks). `make seed-uroojj-full`.
- **Read-only Parameters surface.** Brands, categories, UoMs, locations, tax codes, bank accounts.

## 3. Planned but missing
- **Two of three companies are documentation only — zero code.** No models, routers, or migrations exist for:
  - **Software house** (CRM/clients, projects, timesheets, payroll, HR) — `docs/tenants/software-house.md`, Phase 3.
  - **Religious school + hostel / madrasa** (student registration, guardians, hostel allocation, attendance, fees) — `docs/tenants/religious-school.md`, Phase 4.
  - **HR / payroll engine** — `docs/domain-modules.md` matrix, not built anywhere.
- **Tax engine on posting** — schema exists but no automatic tax line on sales/purchase POST (implementation-status row 10, "partial").
- **Company switcher UI + per-feature RBAC** — UI auto-routes to the first company per org; `auth.py` resolves only `org_id` (no `company_id` in session yet); `db.py` sets only `app.org_id` GUC, so company-level RLS predicate is designed but not yet exercised at runtime (row 11/12, "partial").
- **Parent consolidation** — `parent_consolidation` role designed, not exposed (Section D #13).
- **Document void / reverse / re-issue** — Phase 1.3.F, "next."
- **Bilingual (Urdu) rendering** — schema supports `name_ur`; UI rendering is Phase 1.4, queued.
- **Cloud deploy, backups/DR, custom domains** — Phase 1.5, queued; everything is local today.
- **Billing / subscriptions / pricing / onboarding wizard** — `docs/saas-pricing-and-onboarding.md` is an explicit stub ("PLACEHOLDER"); even the SaaS company name and domain are placeholders (`hamdan-erp.com`).
- **Migration-as-a-service** (`.accdb` → Postgres importer) — described as the blueprint offering; not built.

## 4. The AI gap
**There is no AI in the product today — none.** A repo-wide grep for AI/LLM/agent/forecast/anomaly/NL terms across `apps/api/src` and `apps/web/src` returns a single hit: a WhatsApp *stub* comment in `config.py`. Specifics:
- **"AI Readiness" is a foundation claim, not a feature.** implementation-status row 17 marks it "shipped (foundation)" — meaning a structured `event_log`, stable IDs, and documented query views exist so AI *could* later be plugged in. No model, no inference, no agent.
- **Every AI capability is Section D roadmap (unbuilt):** NL Q&A reporting, anomaly detection, auto-narrative P&L, per-SKU demand forecasting / reorder points / dead-stock alerts, AI-assisted bank reconciliation, and "AI Agents in Place of Additional Staff" (bank-rec agent, AR-chase agent, PO-draft agent, customer-service agent). All listed as separately-priced future engagements in `docs/proposal/Al-Hamdan_ERP_Proposal.pdf` Section D.
- **WhatsApp connectivity** (the channel those agents would use) is a deferred no-op adapter per `docs/tech-stack.md`.

Net: this is a rule-based, trigger-enforced accounting kernel. Its honesty is a plus — it does not pretend the AI is shipped — but for an AI-native showcase, the AI layer is 100% to-be-built.

## 5. Missing pieces to make it sellable
- **Billing & subscription** — no Stripe/Paddle/manual-invoicing path; tiers are a sketch. Blocks selling to a second org.
- **Self-serve onboarding** — wizard ("create your first Company," CoA template per industry, legacy import) is sketched only.
- **Multi-company runtime** — company switcher + per-company RBAC + company-level RLS context must be live before a true holding-company sells.
- **Cloud deployment + DR + custom domains** — currently un-deployable as SaaS; no backups outside future Neon PITR.
- **Tax/compliance completion** — FBR/GST tax engine on documents; FBR e-filing is Section D.
- **Packaging** — define the "base offering" (Section B, 17 rows) as a shippable SKU vs the 14 Section D add-ons; price them.
- **Demo data & demo environment** — good seed exists locally, but no hosted demo/sandbox URL.
- **The non-retail verticals** — software-house and school modules need to exist for the "ERP for any holding company" pitch to be real; today it's an accounting+retail product.

## 6. Native-AI + Autonomous-FTE upgrade
The kernel is an unusually strong substrate for autonomous agents: every posting emits a structured `event_log` event, documents have stable IDs + `source_document_type/id`, money is exact `NUMERIC`, and reporting views expose clean financial state. Proposed FTE roles (the proposal already imagines four):

- **AI Accountant / Bookkeeper** — drafts and posts routine journal entries, classifies transactions to CoA leaves, runs month-end checks (trial-balance-balances, period-lock readiness), flags anomalies. Guardrails: writes only via existing posting services so DB triggers still enforce Dr=Cr, immutability, and period locks; agent proposes, a human approves postings above a threshold. *Replaces ~80–120 hrs/mo of a junior bookkeeper.*
- **AI Bank-Reconciliation Clerk** — ingests bank statement lines (`bank_statements` already modeled), matches to payments/invoices, proposes reconciliations and the adjusting entries for fees/FX. Guardrail: match confidence score; auto-clear only exact matches, queue the rest. *Replaces ~30–50 hrs/mo.*
- **AI Inventory / Reorder Clerk** — watches `stock_valuation_layers` + sales velocity, forecasts per-SKU demand, raises reorder suggestions and dead-stock alerts, drafts purchase orders. Guardrail: drafts only; buyer confirms. *Replaces ~20–40 hrs/mo.*
- **AI AR-Chase / Collections Agent** — reads `v_ar_aging`, sequences reminders over WhatsApp/email, logs promises-to-pay. Guardrail: tone/template allow-list, no balance changes. *Replaces ~20–30 hrs/mo.*
- **AI Reporting Analyst** — NL Q&A over the reporting views, auto-narrative P&L and variance commentary, scheduled WhatsApp/email report packs. Guardrail: read-only over `security_invoker` views (RLS still applies), citations to source rows.
- **(Phase 4 substrate) AI Admissions/Records Officer** — once the madrasa module exists, an agent that digitizes paper registers via OCR + validation into the strongly-validated student schema.

Cross-cutting guardrails: all agents run inside the same RLS/role sandbox (`hamdan_app`, org+company scoped), mutate state only through the audited posting services, and every action lands in `event_log` / `audit_log` for replay.

## 7. Showcase angle (for the portal)
- **Headline:** "The books that close themselves — an AI finance back-office for multi-entity SME groups."
- **Stat-benefit bullets** (aspirational = targets, not shipped):
  - *Target:* an AI bookkeeper + bank-rec clerk that absorbs ~150 hrs/month of finance-team work across a holding group.
  - *Built today:* a rupee-accurate double-entry kernel where every sale, receipt, purchase, and payment auto-posts a balanced, immutable journal entry — verified by a clean trial balance.
  - *Target:* ask your books a question in plain Urdu or English and get a cited P&L narrative — built on the live event log + reporting views that already exist.
- **Demo hook:** Sell a bottle of perfume in the browser and watch the system auto-build the stock issue, COGS at weighted-average cost, and a balanced journal entry — then (roadmap) let the AI accountant explain the month's numbers.

## 8. Verdict
- **AI-native score:** 1/5 — zero AI shipped; only an "AI-ready" event-log/IDs/views foundation. Honest about it.
- **FTE-fit score:** 4/5 — the audited, trigger-enforced, event-emitting accounting kernel is a near-ideal sandbox for autonomous finance agents, and the proposal already scopes four of them.
- **Maturity:** MVP (production-grade kernel; one vertical; local-only; no billing).
- **Recommendation:** **Secondary** — a genuinely solid, sellable accounting/ERP foundation and the best substrate in the portfolio for layered autonomous-FTE agents, but it needs the AI agent layer (plus deploy + billing) built before it can headline the "Restless Workforce" story.
