# Deal Cracker (Card BIN Deal Tracker) — Mobile BIN lookup so restaurant staff apply the right bank-card discount in seconds

## 1. Snapshot
- **One-liner:** A mobile-first webapp that lets restaurant order-takers type a customer's card BIN (first 4-8 digits) and instantly see which active bank-card promotional discount applies today — plus an admin console to manage those bank "alliance" deals.
- **Category:** Fintech (adjacent — promotional/merchant-side discount lookup keyed on card BINs; **not** payments infrastructure, fraud, or transaction routing).
- **Target buyer:** Restaurant chains running co-branded bank card promotions. Concretely built for **Nando's Pakistan** — branding, real Pakistani bank logos (HBL, UBL, Askari, Bank Alfalah, Meezan/Islamic banks, UnionPay, etc.), and a public QR "offers" one-pager all point to a single named client.
- **Tech stack:** Next.js 16 (App Router) · React 19 · TypeScript · Tailwind v4 · Drizzle ORM · Neon serverless PostgreSQL · better-auth (Google OAuth) · deployed on Vercel. ~9,000 LOC of TS/TSX.
- **Maturity:** **MVP, deployed (early production)** — justification: live on Vercel (`card-bin-tracker.vercel.app`), real schema pushed to Neon, Google sign-in with admin email whitelist enforced in middleware, per-user page-view analytics, and **seven** seed scripts importing real bank-alliance data from client Excel/PDF sources spanning Ramadan, Eid, and regular quarterly windows (V2 → V7, Jul–Sep 2026). It's small-scope and single-location, but it is a working product with a real client, not a prototype.
- **Live URL / demo:** `https://card-bin-tracker.vercel.app` (app); public offers page at `/menu/offers`, intended to be reverse-proxied to `https://offers.nandospak.com` (rewrite wired but noted as "if/when" in `next.config.ts`).
- **Path:** /home/anjum/dev/card_bin_tracker

## 2. What's built today (verified)
- **Staff mobile app** (`src/app/user/*`) — 3-tab bottom-nav (Deals / Search / Banks) with a persistent date selector defaulting to today:
  - **Deals drill-down** (3 levels): active deals list → deal detail (bank+network+tier cards with discount %) → BIN list per card entry.
  - **BIN search** (`src/app/api/deals/search/route.ts`): matches on prefix in both directions (`bin LIKE binNumber%` and reverse), joins bank/network/tier, and flags `isActiveToday` by checking date range + weekday. Min 4 digits.
  - **Browse by bank** with real bank logos (`src/lib/bank-logos.tsx`, `public/logos/*`).
- **Admin console** (`src/app/admin/*`, desktop) — full CRUD for deals (create/edit form with weekday picker, discount cap, card type) and reference data (banks, card networks, card tiers). REST routes under `src/app/api/{deals,banks,networks,tiers}`.
- **Data model** (`src/db/schema.ts`): `deals` (date range + `validDays[]` weekday array) → `dealEntries` (bank × network × tier × discount% × optional PKR cap × credit/debit) → `dealEntryBins` (BIN + original card product name). Plus `pageViews` for analytics.
- **Auth & access control** (`src/middleware.ts`, `src/lib/auth/*`): better-auth Google OAuth; unauthenticated users redirected to `/login`; `/admin` gated by `ADMIN_EMAILS` whitelist.
- **Analytics** (`src/app/api/analytics`): admin-only dashboard — total/unique/today views, per-page and per-user breakdowns, page-view tracking hook.
- **Public customer-facing offers one-pager** (`src/app/menu/offers`, `/api/public/active-deals`) — Nando's-branded, cached (5-min ISR), built for QR tent cards.
- **Real data ingestion**: seed scripts `seed.ts` … `seed-v7-jul-sep.ts` parse client Excel via the `xlsx` lib, with **hardcoded heuristics** (`deriveNetwork`, `deriveTier`) that infer Visa/Mastercard/UnionPay/PayPak and tier (Gold/Platinum/Signature/Infinite…) from BIN prefix ranges and card-name keywords. This is rule-based string matching, not ML.

## 3. Planned but missing
Per `specs/001-bin-deal-tracker/spec.md` "Out of Scope" and observable gaps:
- Multi-location / multi-restaurant support (single location assumed).
- POS / payment-processing integration (lookup only; no transaction is touched).
- External BIN database validation or auto-lookup — admins are trusted to enter correct BINs.
- Deal-usage / redemption reporting and analytics (only page-view analytics exist).
- Notifications for expiring deals.
- Offline / PWA functionality.
- In-app deal data entry from source files: today new deal periods require an engineer to write/run a new `seed-vN` TypeScript script, not an admin upload flow.

## 4. The AI gap
**There is no AI in this product. Zero LLM, agent, embedding, or ML dependency** — `package.json` has no `@anthropic-ai`, `openai`, or any model SDK; nothing calls a model anywhere in the tree. The only "intelligence" is a deterministic if/else heuristic (`deriveNetwork`/`deriveTier`) that maps BIN prefixes and card-name keywords to networks/tiers during seeding.

The real, AI-shaped pain hides in the workflow *around* the app: every quarter (and for each Ramadan/Eid campaign) a human reads bank "alliance" PDFs and multi-sheet Excel files, normalizes inconsistent bank/card/tier naming, extracts hundreds of BINs, and a developer hand-codes a seed script. That ingestion-and-normalization loop — currently manual and engineer-dependent — is exactly where an agent belongs. The lookup app itself is a thin, correct CRUD/search layer with no model in sight.

## 5. Missing pieces to make it sellable
- **Self-serve deal ingestion**: admin uploads the bank's Excel/PDF and the system extracts deals/BINs/discounts — removing the engineer-writes-a-seed-script bottleneck (the single biggest operational cost today).
- **BIN data quality**: integration with a real BIN/IIN reference (network, issuer, country, card brand) to validate admin entries and catch typos/conflicts before they cause a wrong discount at the till.
- **Multi-tenant**: per-brand/per-location config so it can be sold beyond one Nando's market.
- **Redemption analytics**: which deals/BINs are actually being looked up and applied — the value story for the restaurant CFO.
- **Conflict/overlap handling UI**: surface when one BIN maps to multiple active deals (schema allows it; staff UX should disambiguate).
- **Hardening**: tests (none found), audit log on admin edits, and a documented onboarding flow.

## 6. Native-AI + Autonomous-FTE upgrade
Reframe the product around an **AI Deal-Ops Analyst (autonomous FTE)** that owns the bank-alliance pipeline 24/7:
- **Ingest**: watches an inbox/drive for the bank's alliance PDF/XLSX, parses sheets, and extracts deal name, validity window, valid weekdays, bank, network, tier, discount %, PKR cap, and the BIN list — handling the messy, inconsistent layouts that currently need a developer.
- **Normalize & enrich**: resolves "Bank Al-Habib" vs "BAHL", maps card products to canonical network/tier, validates each BIN against a real IIN database, and flags BINs that don't match the claimed network/bank.
- **Reconcile**: diffs against live deals, proposes adds/edits/expiries, and surfaces overlaps/conflicts for one-click human approval.
- **Publish & monitor**: pushes approved deals to Neon, refreshes the public QR offers page, and answers staff/manager questions ("what's the best HBL credit deal this Friday?").
- **Replaces ~15–25 hours/month** of a payments/marketing ops analyst's manual Excel-wrangling and the recurring engineer time spent writing seed scripts each campaign — call it **~0.15 FTE of analyst + recurring dev hours eliminated**.
- **Guardrails**: human-in-the-loop approval before any deal goes live; every BIN validated against an external reference; full audit trail of agent-proposed vs admin-confirmed changes; no autonomous edits to active deals without sign-off (a wrong discount is real money at the counter).

## 7. Showcase angle (for the portal)
- **Headline:** "From bank PDF to the till in minutes — an AI Deal-Ops analyst that keeps every card discount live, validated, and instant for staff."
- **Stat-benefit bullets:**
  - **<1s** card-to-discount lookup for order-takers — type the BIN, see the deal, charge correctly.
  - **~0.15 FTE + recurring dev time reclaimed** by automating quarterly bank-alliance Excel/PDF ingestion that's manual today.
  - **7 campaign datasets, hundreds of BINs, real banks** already modeled — a working, deployed foundation, not a slide.
- **Demo hook:** Drop a bank's alliance Excel into the app, watch the agent extract, normalize, and validate hundreds of BINs into live deals — then scan the QR and search a card BIN on a phone to see the discount appear instantly.

## 8. Verdict
- **AI-native score:** 0/5 — genuinely no AI today; only rule-based heuristics. The opportunity is large but entirely greenfield.
- **FTE-fit score:** 3/5 — the surrounding ingestion/normalization workflow is a clean, real, recurring analyst+dev task that an agent could own with human approval; the lookup app itself isn't FTE-shaped.
- **Maturity:** MVP / early production — deployed, authenticated, real client data across 7 campaigns, but single-tenant, untested, and engineer-dependent for data entry.
- **Recommendation:** **Incubate.** Polished, real, and demo-friendly with a credible named client (Nando's Pakistan), but it ships zero AI and is narrowly scoped to one merchant. Worth incubating specifically to build the AI Deal-Ops ingestion agent — that upgrade is what turns a tidy CRUD app into a sellable autonomous-FTE story; without it, it's a nice internal tool, not a flagship.
