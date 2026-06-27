# PromptForge — A polished template-driven prompt builder with optional one-shot LLM passthrough

## 1. Snapshot
- **One-liner:** A full-stack web app where a single user browses curated productivity prompt templates, fills in parameters, generates a finished prompt by string substitution, and optionally fires it once at any of 11 LLM providers.
- **Category:** AI Productivity / Tooling (prompt management + thin LLM gateway)
- **Target buyer:** Individual knowledge workers, solopreneurs, and "prompt-curious" professionals who want better-structured prompts without writing them from scratch. Not yet a team/enterprise product (no multi-user).
- **Tech stack:** Backend — FastAPI (Python 3.12, async), SQLModel ORM, PostgreSQL 16 (Docker), httpx, `google-genai` SDK, pydantic-settings, Uvicorn. Frontend — Next.js 16 (App Router, TypeScript strict), React 19, Tailwind v4, react-icons. Infra — docker-compose (Postgres only).
- **Maturity:** **MVP.** Justification: both tiers are actually built and wired end-to-end (browse → fill → generate → send → history persists). It is single-tenant, has no auth, stores API keys in plaintext, has CORS wide open, ships no tests, and has no deployment — so it is a working MVP, not production. Note: CLAUDE.md and the spec call the frontend "scaffolded only," but the code shows a fully implemented UI (12 components + typed API client) — the docs lag the actual state.
- **Live URL / demo:** None. Localhost only (frontend :3000, API :8000, DB :5433). No hosting config, no Dockerfile for the app tiers, no CI.
- **Path:** /home/anjum/dev/prompt_creator

## 2. What's built today (verified)
**Backend (genuinely complete):**
- FastAPI app (`backend/app/main.py`) with lifespan table-creation, 5 routers: health, prompts, llm, settings, history.
- Prompt CRUD + filtering by category/search (`routers/prompts.py`, `services/prompt_service.py`).
- Template generation = literal string substitution: `template.replace("[key]", value)` (`prompt_service.fill_template`). No AI involved in "generate."
- LLM passthrough (`services/llm_service.py`): three API formats — OpenAI-compatible, Anthropic Messages, Google Gemini via `google-genai`. 11 providers mapped in `services/llm_providers.py` (openai, anthropic, google, meta/together, mistral, groq, cohere, deepseek, perplexity, xai, ollama + custom). Gemini path includes multi-API-key round-robin rotation with 429/quota fallback.
- Settings: single global `LLMConfig` row (`name="default"`, one `is_active`); API keys masked to last-4 in responses (`routers/settings.py`).
- History: every `/api/llm/chat` call persists filled prompt, params, response, provider, model, tokens, latency (`routers/history.py`, `models/prompt_history.py`); list + delete endpoints.
- 4-table schema (prompts, prompt_parameters, llm_configs, prompt_history). Seed script loads the catalog (`backend/scripts/seed.py`).

**Frontend (built, contrary to the docs):**
- Full single-page app (`frontend/src/app/page.tsx`) with Navbar, CategoryFilter, SearchBar, PromptGrid/PromptCard, PromptBuilder, ParameterForm (text/textarea/select/slider), PromptOutput (generated prompt + AI response tabs), SettingsDrawer (provider cards, model dropdown, masked key, test-connection, temp/max-tokens/top-p), PromptHistoryList ("My Prompts"), Footer.
- Typed API client (`frontend/src/lib/api.ts`, `types.ts`) covering every endpoint. Dark-mode glass-morphism theme per the UX spec.

**Template library:** Catalog ships **35** templates (`data/prompts-catalog.json`), not the "20" claimed in CLAUDE.md/spec. Categories span Productivity, Writing & Editing, Professional Communication, Prompt Mastery, Health & Energy, Finance, Relationships, Routines, etc. Each has 3–7 parameters.

**Auth:** **None.** No user model, login, session, or tenancy anywhere in the backend.

## 3. Planned but missing
- Spec `001-promptforge-frontend` exists with full FR/acceptance criteria; its `tasks.md` lists every task as **Status: pending** (0 checked) even though the UI is built — task tracking was never updated.
- Explicitly out-of-scope in the spec (and absent): authentication/multi-user, real-time LLM streaming, custom-prompt creation UI (API-only today), light mode, E2E tests, mobile.
- No deployment/hosting, no app Dockerfiles, no CI, no test suite (backend or frontend).
- Docs/reality drift: "frontend scaffolded only," "20 prompts" (actually 35), and a known `.env` port mismatch (5432 vs 5433) all called out but unreconciled.

## 4. The AI gap
This is a **prompt-formatting tool with an optional thin LLM gateway**, not an AI product.
- The core "generate" action contains **zero AI** — it is `str.replace` on a human-authored template.
- The only AI is an **optional, single-shot, fire-and-forget** call to a third-party model the *user* must configure with *their own* API key. No model is bundled, fine-tuned, or owned.
- No chaining, no multi-step reasoning, no tool use, no retrieval, no memory, no agents, no evaluation/scoring of outputs, no scheduling, no autonomy. One human, one click, one response.
- The Gemini path truncates prompts to 4000 chars for quota management — a silent quality risk that underlines this is a budget passthrough, not a reasoning engine.
- "Prompt Mastery" templates (e.g. "Master Prompt System Designer") gesture at sophistication but are still static fill-in-the-blank text.

## 5. Missing pieces to make it sellable
- **Auth + multi-tenancy + key vaulting:** real users, per-user/per-org isolation, encrypted secrets (currently plaintext keys, single global config, CORS `allow_origins=["*"]` — a security non-starter for any paid deployment).
- **Deployment story:** hosted URL, app containers, CI/CD, env management.
- **A reason to pay:** today it competes with free prompt galleries. Needs owned value — streaming, output quality scoring, prompt versioning/A-B testing, team sharing, analytics on what works.
- **Tests & reliability:** zero automated coverage; the LLM gateway (the riskiest surface) is untested.
- **Trust/compliance:** usage limits, audit log, PII handling, rate limiting.

## 6. Native-AI + Autonomous-FTE upgrade
Reframe PromptForge from "a person crafts one prompt at a time" to **"a person designs and deploys autonomous AI workers."** The existing assets map cleanly:
- **Templates → deployable agents.** Each template already has a parameterized task definition. Promote it to a saved "Worker" with a goal, inputs, an attached model/provider, and a schedule/trigger. The catalog of 35 becomes a starter library of FTE job descriptions (a "Writing Editor FTE," a "Weekly Planning FTE," a "Financial Pulse FTE").
- **One-shot chat → autonomous loops.** Replace fire-and-forget with run loops: input source (webhook/queue/cron) → fill template → call model → tool/action step → evaluate → persist → notify. The `prompt_history` table is already a run log; extend it into a job/run ledger.
- **"Lets one person run N AI workers."** Add a worker dashboard: N concurrent agents, each with its own config, schedule, and output stream — the operator supervises instead of types.
- **Guardrails (net-new, required):** per-worker budgets/token caps (the truncation hack hints at the need), output validators, human-in-the-loop approval gates, kill switch, and an audit trail. The multi-key rotation already shows quota-awareness to build on.
- **Provider abstraction is a real asset:** the 3-format / 11-provider gateway is exactly the routing layer an FTE platform needs (cost/latency/fallback routing across models).

## 7. Showcase angle (for the portal)
- **Headline:** "From prompt to payroll — turn a template into an AI worker that runs itself."
- Stat-benefit bullets:
  - **35 ready-to-deploy job templates** — staff a writer, planner, or analyst in one click instead of hiring.
  - **11 model providers, one control panel** — route every AI worker to the cheapest/fastest brain with automatic key-rotation failover.
  - **Every run logged and costed** — full history of outputs, tokens, and latency, so one operator can supervise many workers with receipts.
- **Demo hook:** Pick "Weekly Planning," fill three fields, and instead of a single answer, deploy it as a recurring Monday-morning AI worker that emails you the plan — show the same UI doing one-shot today and scheduled-autonomy tomorrow.

## 8. Verdict
- **AI-native score:** 2/5 — clean multi-provider gateway and a real product surface, but the "AI" is an optional one-shot passthrough on string-substituted templates; core value is template management, not intelligence.
- **FTE-fit score:** 1/5 — entirely human-in-the-loop, single-shot, single-user; nothing autonomous, scheduled, or 24/7. The architecture *could* host it, but none of it exists today.
- **Maturity:** MVP (working full-stack, single-tenant, no auth/deploy/tests).
- **Recommendation:** **Incubate.** The UI polish, provider abstraction, and run-history schema are a genuinely good foundation, but it is two pivots away from the autonomous-FTE thesis. Fund the "templates → deployable workers + guardrails" reframe; do not showcase it as shipped autonomous AI.
