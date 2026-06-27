# ComplaintHub — The unified, omnichannel complaint system of record, live in production

## 1. Snapshot
- **One-liner:** Corporate complaint management platform that captures, routes, tracks and auto-resolves complaints across every channel (web, email, WhatsApp) with a 100% audit trail.
- **Category:** CX / Helpdesk / Complaint & Case Management (vertical: multi-location consumer brands)
- **Target buyer:** Multi-location operators — restaurants, retail, franchises — and SMEs/corporates needing one unified complaint system across business units and regions.
- **Tech stack:** Next.js 16 (App Router, TS strict, Turbopack), Neon Postgres (us-east-1, serverless), Zod validation, bcryptjs + HTTP-only cookie auth, RBAC middleware, Recharts analytics, a Baileys-style WhatsApp bridge (PM2 `whatsapp-bridge` :8080), Gmail OAuth email intake. Deployed on Oracle A1.Flex (Always Free, $0/mo) behind cPanel `.htaccess` proxy, PM2-managed (`cms-app` :3000).
- **Maturity:** **Production.** Justification: live and serving real users at https://nandospak.com/cms/ for Nando's Pakistan; full deploy infra (deploy.sh/install.sh, nginx, PM2 ecosystem), a self-updating local replica with in-app App Management/version-detect, hardened security baseline, migrations on Neon, and operational runbooks (WhatsApp re-pair, A1 capacity retry). This is the most mature product in the portfolio and the reference anchor.
- **Live URL / demo:** https://nandospak.com/cms/
- **Path:** /home/anjum/dev/cms

## 2. What's built today (verified)
Genuinely shipped and running in production:
- **Omnichannel intake** — web form, email (Gmail OAuth), and WhatsApp via a dedicated bridge process. 3+ channels into one queue.
- **Rule-based routing** — deterministic routing by category / subcategory / location to the right owner/team. (`lib/` business logic; counter-positioned in marketing as "intelligent rule-based routing.")
- **Configurable SLA auto-close** — `lib/auto-close.ts` drives time-based auto-resolution with configurable triggers (marketed as 9 triggers).
- **RBAC** — role-guarded middleware (`src/middleware.ts`), super-admin gating via `requireSuperAdmin()`, HTTP-only cookie sessions, bcryptjs hashing, Zod on every input.
- **100% audit trail** — full system of record; remarks/notifications subsystems (`lib/remarks.ts`, `lib/notifications.ts`).
- **Analytics dashboard** — Recharts-based reporting over complaint data.
- **Unified system of record** — single store for all complaints across channels/locations.
- **WhatsApp bridge** — separate PM2 service with QR-pairing admin flow and re-pair runbook.
- **Deploy/infra** — one-command deploy to Oracle, $0/mo Always-Free hosting, self-updating WSL replica with in-app update detection (version-bump convention), Neon serverless DB.
- **Two complete marketing kits** — consumer (warm/human) and enterprise (AI-native) landing pages + reveal.js decks sharing a design system.

## 3. Planned but missing
- **Multi-tenant SaaS** — currently single-tenant (Nando's Pakistan); no tenant isolation/self-serve provisioning.
- **Self-serve onboarding** — deployment is operator-led (deploy scripts), not "start free" sign-up despite PLG CTAs in the copy.
- **Billing / subscription** — no metering or payment layer ("$0 infra / no per-agent tax" is positioning, not a billing engine).
- **Predictive analytics** — reporting is descriptive (Recharts), not forecasting.
- **The full AI layer** (see section 4).

## 4. The AI gap
This is the honest core. The enterprise marketing kit sells an "AI-native operating system" — intelligent triage, sentiment, AI copilot, predictive analytics — but **none of that is wired in the shipped product.** Per the marketing kit's own honesty note: the product is *largely rule-based* (routing by category/subcategory/location, configurable auto-close SLA, Recharts analytics). **Sentiment-analysis UI scaffolding exists but isn't fully wired.** AI triage / sentiment / copilot / predictive analytics must be treated as **roadmap/vision**, and the headline stats (70%, 3×) as **illustrative targets, labeled as such** — never demoed as live. What is genuinely shipped: omnichannel intake, rule-based routing, 100% audit trail, RBAC, unified system of record. Rule: *don't present roadmap AI as live in a deal you can't demo.* The two strongest selling motions today are honest counter-positioning (real human owners + WhatsApp speed, against the AI-bot crowd) and the unified system-of-record + ownership ($0 infra, own your data) angle.

## 5. Missing pieces to make it sellable
Already sellable and live — focus is selling *beyond* Nando's:
- **Multi-tenancy** — tenant isolation, per-tenant config/branding, row-level data separation.
- **Self-serve onboarding** — sign-up → provision → import categories/locations, to honor the PLG "Start free" CTA.
- **Billing** — subscription/metering (seats or volume), so "no per-agent tax" becomes a real pricing model.
- **The AI layer** — convert the scaffolding into shipped triage/sentiment/copilot (section 6) so the enterprise kit's claims become demonstrable rather than aspirational.
- **Reference proof** — replace illustrative capability stats with named-customer quote + hard ROI number (Nando's Pakistan is the obvious first case study).

## 6. Native-AI + Autonomous-FTE upgrade
Layer autonomous agents on the existing system of record (the audit trail + unified store make this low-risk to bolt on):
- **AI complaint-triage agent (24/7):** classifies category/subcategory/severity, detects sentiment/urgency, and routes — replacing manual triage and upgrading today's static rules with judgment. Falls back to deterministic rules on low confidence.
- **AI resolution copilot:** drafts responses, suggests resolution steps from similar resolved cases, surfaces the SLA clock — accelerating each handler.
- **AI CX analyst:** continuously reads the complaint stream, flags emerging issues (a bad batch, a problem location), and produces weekly trend briefs — turning descriptive Recharts into proactive intelligence.
- **FTE framing:** "Replaces ~1–2 FTEs of a CX/complaints triage team" — ~16–24 person-hours/day of reading, tagging, routing, and first-draft replies, running around the clock.
- **Guardrails:** human-in-the-loop approval before customer-facing sends, confidence-thresholded rule fallback, 100% audit trail on every agent action, RBAC-scoped agent permissions, no autonomous case closure without SLA/policy check.

## 7. Showcase angle (for the portal)
- **Headline:** "Every complaint, every channel, one system of record — live in production at $0 infra."
- **Stat-benefit bullets:**
  - **Omnichannel by default** — web + email + WhatsApp into one unified queue (3+ channels, no silos).
  - **100% audited** — every complaint and action on a full audit trail; RBAC-secured system of record.
  - **$0 infra, in production today** — running live for a national brand on Always-Free cloud; own your data, no per-agent tax.
- **Demo hook:** File a complaint via WhatsApp and watch it route, track, and auto-resolve on the live dashboard — then show the roadmap AI-triage agent classifying it in real time.

## 8. Verdict
- **AI-native score:** 2/5 — low today (rule-based, sentiment only scaffolded), but high potential: a clean system of record + audit trail is the ideal substrate for an agentic layer.
- **FTE-fit score:** 4/5 — complaint triage/routing/first-response is a textbook autonomous-FTE workload, and the platform already owns the data and workflow.
- **Maturity:** Production.
- **Recommendation:** **Flagship** — the only live, production, real-customer product in the portfolio; proven infra and honest positioning make it the credibility anchor for the whole showcase.
