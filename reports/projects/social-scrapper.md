# Sentinel CX — An always-on AI brand-reputation analyst that reads every review, scores it, and drafts the reply

## 1. Snapshot
- **One-liner:** Multi-platform social/review monitor that scrapes customer mentions, classifies intent & severity with an LLM, drafts a suggested reply, and pings a manager on WhatsApp/Email — built for Nando's Pakistan, generalizes to any multi-location CX/brand-reputation product.
- **Category:** CX / Social Listening (review & reputation management)
- **Target buyer:** Multi-location F&B / retail brands; CX, marketing-ops, and social/community-management teams who today manually triage Google/Facebook/Instagram reviews.
- **Tech stack:** Backend — Python 3.11, FastAPI (async), SQLModel + asyncpg, PostgreSQL, Alembic, tenacity, Google Places API, **Google Gemini (`gemini-2.5-flash`)** for classification, Gmail OAuth API (email out), WhatsApp via external Go bridge (whatsapp-mcp). Frontend — Next.js 14 (App Router), TypeScript, Tailwind + shadcn/ui, TanStack Query, Plotly.js. Infra — Docker Compose (Postgres), SSE for live progress.
- **Maturity:** **MVP (single-platform).** Justification: one real end-to-end pipeline works (scrape Google Reviews → store → Gemini classify → draft reply → notify), backed by a real DB, migrations, and a partly-wired dashboard. But only 1 of 5 promised platforms is implemented, the approval/escalation workflow is schema-only, large parts of the dashboard are hardcoded mock data, there is no auth, no deployment, and API keys are committed to the repo. Above prototype, well below production.
- **Live URL / demo:** None. Local only (`localhost:3000` frontend, `localhost:8000` API, Postgres via docker-compose). Tagged `v1.0.0-fte-milestone`; spec marks "Deploy to staging" as still pending.
- **Path:** /home/anjum/dev/social_scrapper

## 2. What's built today (verified)
Verified by reading source, not docs:

- **Google Reviews scraper (the only working scraper).** `backend/app/scrapers/google_reviews.py` — fetches reviews via Google Places Details API across a comma-separated list of Place IDs (16 Nando's locations enumerated in `notes.md`), dedupes on `platform_post_id`, writes `Post` rows, records a `ScraperRun`, exponential-backoff retry (tenacity), a 500-post in-memory buffer for DB outages, and an SSE `scrape_with_progress` stream with per-location events. Note: Google Places returns at most ~5 reviews per location per call — a hard ceiling on data freshness/volume.
- **Real LLM classification + reply drafting (Gemini).** `backend/app/services/classification_service.py` — prompts Gemini to return JSON with `intent` (complaint_severe/moderate/mild, praise, question, neutral, suggestion), `severity`, `confidence`, `reasoning`, and a `suggested_reply` for every post. Production-grade resilience: 5-key rotation with quota-aware fallback, 3 retries/key with exponential backoff, markdown-strip, and a regex-based truncated-JSON repair. Results persisted to `classifications` table.
- **Notification fan-out.** `whatsapp_service.py` (POSTs formatted message to the local Go bridge REST API) and `email_service.py` (Gmail OAuth). Both fire only when a `suggested_reply` exists, with duplicate-prevention via `whatsapp_notifications` / `email_notifications` tables. Disabled by default (env-gated).
- **"Autonomous FTE Mode" (browser-driven).** `frontend/app/page.tsx` + `components/dashboard/autonomous-mode.tsx` — a countdown timer that, on each interval, calls scrape → classify → refresh via SSE. Important caveat: this loop runs in the open browser tab (`setInterval`), not as a server daemon. A backend `BaseScraper` background loop (`start()`/`_scrape_loop`) exists but the shipped "FTE" UX is tab-bound, so "24/7" requires a browser left open.
- **REST + SSE API.** `posts`, `classification` (batch / real-posts / selected, all with live SSE progress + per-key telemetry), `scrapers` (run / run-stream / start / stop / stats), `whatsapp`, `seed`, `health`.
- **Database & migrations.** Postgres via SQLModel; tables: `posts`, `classifications`, `scraper_runs`, `draft_responses`, `escalations`, `whatsapp_notifications`, `email_notifications`. 4 Alembic migrations present.
- **Dashboard (partly real, partly mock).** Real & wired: post list with filters/search/pagination/delete, DEMO/REAL toggle, scraper control, classification control with streaming logs. **Hardcoded mock data:** `analytics-charts.tsx` (fixed `volumeData`), `stats-overview.tsx` (fixed sparklines), `response-templates.tsx` (3 static templates), plus Team Activity / Performance Metrics / Platform Performance and the "Refresh/Export/Advanced Filters" buttons and "Last updated: 2 minutes ago" label — all placeholder.
- **Tests.** ~412 lines across unit (Google Reviews scraper, mocked) and integration (scraper API) tests + fixtures; spec shows MVP-phase tests done, several unit tasks still open. No coverage of classification or notification services.

## 3. Planned but missing
Specified (`specs/002-social-media-scrapers`) and scaffolded but **not implemented** — all marked `[ ]` pending:
- **Facebook scraper** (US3/P2, tasks T049–T062), **Instagram scraper** (US4/P2, T063–T076), **Email ingestion scraper** (US5/P3, T077–T090), **Yumpingho scraper** (US6/P3, T091–T105, blocked on a ToS review). All five platforms exist in the `Platform` enum and `config.py` intervals, but only Google Reviews has code. So "5-platform monitor" is 1 of 5 shipped.
- **Human approval / response-management workflow.** `DraftResponse` (pending/approved/rejected, approved_by, rejection_reason) and `Escalation` (severity, status, email_recipient, resolution_notes) models exist but are **never written** — only read in `post_service.py`. There is no endpoint to approve/edit/reject a draft or to escalate. The spec'd Approve/Edit/Reject UI and severe-complaint auto-escalation-to-email are not built.
- **Staging/production deployment** (T118) — pending. No hosting, no CI.
- Constitution lists "OpenAI Agents SDK as orchestration" and pluggable LLM providers as **future** direction — not present.

## 4. The AI gap
- **The AI that exists is real and is the product's best asset — but it is Google Gemini, not Claude/Anthropic.** Classification + reply drafting is a genuine LLM call (`google-genai`, `gemini-2.5-flash`) with serious operational hardening (key rotation, backoff, JSON repair). This is shipped, not roadmap.
- **It stops at "draft," by design.** The constitution mandates *"ALL responses require human approval before posting"* and *"No auto-posting to any platform."* The system reads and drafts; a human still copies/pastes the reply. So it is an AI *assistant*, not an autonomous *actor* — a meaningful gap versus the "autonomous FTE" pitch.
- **No sentiment scoring beyond categorical intent**, no per-brand tone/voice tuning, no trend/topic clustering, no learning from approved-vs-rejected replies. The "AI" is one stateless prompt per post.
- **The "autonomous" loop is a browser timer, not an agent.** No server-side scheduler is wired into the shipped UX, no agentic planning, no tool-use loop, no memory.
- **Dashboard intelligence is faked.** The analytics/insights a buyer would judge "AI" by (volume trends, sentiment over time, performance) are hardcoded arrays, not derived from data.

## 5. Missing pieces to make it sellable
- **Breadth:** implement at least Facebook + Instagram (and ideally TripAdvisor/Yelp-class review sites) so "multi-platform" is true.
- **Close the response loop:** build the approval queue (wire `DraftResponse`/`Escalation`) and add real write-back/auto-post to Google Business, Meta, etc. with guardrails — this is the actual value (saving reply labor), not just alerting.
- **Real analytics:** replace mock charts with DB-driven sentiment/volume/SLA dashboards; per-location and per-brand breakdowns.
- **Multi-tenancy + auth:** today it is hardwired to one brand ("Nando's"), no login, no roles. A sellable product needs tenant isolation and user accounts.
- **Server-side scheduler/worker** (e.g. APScheduler/Celery) so "24/7" doesn't depend on an open browser tab.
- **Security hygiene:** Google + Gemini API keys and Place IDs are committed in `notes.md`/`social_scrapper_notes.md`; `backend/config/credentials.json` + `token.json` are present. Rotate, remove from history, move to secrets manager.
- **Deployment, observability, and tests** for the AI/notification paths.

## 6. Native-AI + Autonomous-FTE upgrade
Reframe from "alerting tool" to an **AI Community Manager / Brand-Reputation FTE** that runs as a server-side agent:
- **Reads 24/7** across Google, Meta, review sites, and inbox — server-scheduled, no browser dependency.
- **Sentiment + intent + severity scoring** on every mention, with topic/trend clustering and per-location heatmaps.
- **Drafts on-brand replies** using a tunable brand-voice profile, then **auto-posts** within configurable guardrails: auto-publish praise/FAQ replies; hold complaints for one-click human approval; auto-escalate severe (health/safety/legal) cases to a named owner with full context.
- **Closes the loop / learns** from approved-vs-edited replies to improve tone over time; weekly auto-generated reputation digest.
- **Pitch:** "Replaces ~1 FTE of social/CX triage — roughly 30–40 hours/week of reading reviews, sorting urgency, and writing first-draft replies — for a multi-location brand, with a human approving only the sensitive 10%."
- **Guardrails/approval:** confidence + severity thresholds, brand-safety filters, full audit trail per `DraftResponse`/`Escalation`, and a human-in-the-loop queue (the schema already anticipates this — it just needs to be wired). Migrating the LLM layer to Claude with tool-use would turn today's single-shot prompt into a real agentic loop.

## 7. Showcase angle (for the portal)
- **Headline:** "Your reviews, read and answered before your coffee's cold — an AI brand-reputation FTE that never sleeps."
- 3 stat-benefit bullets:
  - **Every review triaged in seconds:** auto-classifies intent + severity and drafts an on-brand reply for 100% of incoming mentions.
  - **Severe issues never sit unseen:** instant WhatsApp + Email escalation to the right manager, with dedupe so no double-pings.
  - **~1 FTE of CX triage offloaded:** ~30–40 hrs/week of reading, sorting, and first-draft writing handled autonomously — humans approve only what matters.
- **Demo hook:** Flip on "Autonomous FTE Mode," watch the live SSE stream pull real Nando's Google reviews, classify each one, and a WhatsApp draft-reply land on a phone — all hands-free.

## 8. Verdict
- **AI-native score:** 3/5 — genuinely AI-powered core (real, hardened Gemini classification + reply drafting), but it's Gemini not Claude, single-shot not agentic, draft-only not acting, and the visible "analytics AI" is mocked.
- **FTE-fit score:** 3/5 — excellent narrative fit and it already brands itself an "Autonomous FTE"; but autonomy is browser-tab-bound, only 1 platform works, and it alerts rather than resolves (no auto-post, approval workflow unbuilt).
- **Maturity:** MVP (single-platform); one real pipeline, lots of mock UI, no auth/deploy, committed secrets.
- **Recommendation:** **Secondary** — compelling story and a real AI core make it a strong showcase demo, but it needs platform breadth, a wired approval/auto-post loop, and a server-side scheduler before it's flagship-grade. Clear, fast path from Secondary to Flagship.
