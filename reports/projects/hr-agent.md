# HR Agent — An always-on AI recruiting screener that ingests, parses, scores, and shortlists candidates while a human approves the calls

## 1. Snapshot
- **One-liner:** AI-powered recruitment automation that auto-collects email applications, parses CVs, and scores candidates against weighted job criteria with explainable reasoning — keeping the HR manager as final approver.
- **Category:** HR Tech / Talent Acquisition (AI applicant screening & shortlisting).
- **Target buyer:** SMB-to-mid-market HR managers / in-house recruiters drowning in inbound CVs; recruiting agencies. Originally scoped for a Pakistan-based single-HR-manager use case (`specs/hr-agent-mvp/project-summary.md`, open question on Pakistan labor law).
- **Tech stack:** Backend FastAPI + Python 3.12, SQLModel/SQLAlchemy async + PostgreSQL, Alembic migrations, Redis (declared, unused in code), MinIO/S3 for CV storage, OpenAI `gpt-4o-mini` for all LLM work, PyMuPDF for PDF text, Gmail API (email intake) + Google Calendar API (availability). Frontend Next.js 16 / React 19 / TypeScript / Tailwind v4 / shadcn-ui / TanStack Query / Recharts. Docker Compose for infra (Postgres/Redis/MinIO).
- **Maturity:** **Prototype** (early MVP backend). Justification: substantial code exists (~3,300 lines of services, ~1,500 lines of API routes, 9 models, 3 Alembic migrations, a full set of frontend pages), but (a) there are **runtime-breaking model/service field mismatches** — `candidate_service.py` and the `CandidateResponse` API schema read `candidate.overall_score`, `ai_summary`, `parsed_data`, `cv_file_path`, and `candidate.scores`, none of which exist on the `Candidate` model (which has `resume_path`, `resume_data`, and a singular `score` relationship); `email_polling_service._create_candidate()` passes the same non-existent kwargs (`parsed_data=`, `cv_file_path=`) — so the candidate list/detail endpoints and the email ingest path would throw at runtime; (b) the `tests/` directory is **empty** despite an 80%-coverage acceptance criterion; (c) backend/frontend Docker images are commented out in `docker-compose.yml` (infra only); (d) no CI, no deployment. It has never been proven end-to-end.
- **Live URL / demo:** None. No deployed instance; frontend builds locally (`.next/` present), backend runs via `uvicorn src.main:app`. A `(dashboard)/demo` page exists but only showcases two UI modals.
- **Path:** /home/anjum/dev/hr-agent

## 2. What's built today (verified)
Backend (FastAPI, `backend/src/`):
- **Auth:** JWT login, bcrypt password hashing, `CurrentUser` dependency, seed script for an initial HR Manager (`api/auth.py`, `utils/jwt.py`, `utils/security.py`, `scripts/seed.py`).
- **Jobs CRUD + criteria:** create/list/get/update/status-change/delete jobs, set weighted + mandatory criteria (`api/jobs.py` — 7 routes; `services/job_service.py`, 464 lines; models `job.py`, `job_criteria.py`).
- **Criteria templates:** pre-built/customizable role templates (`api/templates.py`, `models/criteria_template.py`, migration `1f61449e8b94`).
- **CV parsing (LLM):** PyMuPDF extracts PDF text, then OpenAI prompt returns structured JSON (name, contact, experience, education, skills) validated into a `ParsedCV` Pydantic model (`services/cv_parsing_service.py`). Input truncated to 8,000 chars.
- **Candidate scoring (LLM, explainable):** builds a prompt from parsed CV + job criteria, OpenAI returns per-criterion scores (0–100 weighted / pass-fail mandatory) plus reasoning and an `ai_summary`; results persisted to `candidate_scores` with confidence band and `is_recommended` flag (`services/scoring_service.py`, `models/candidate_score.py`). This persistence path is internally consistent.
- **Email intake pipeline (LLM-classified):** background async loop polls Gmail every 5 min, fetches unread mail, extracts PDF attachments, classifies the email to a job via LLM (or auto-assigns when one active job), uploads CV to storage, parses, creates candidate, and auto-scores (`services/email_polling_service.py`, `services/background_tasks.py`, `providers/email/gmail.py`). Note the candidate-creation kwargs bug above; the polling loop only starts if `gmail_credentials_path` is configured.
- **Calendar availability:** Google OAuth connect/callback/disconnect, fetch events, compute free interview slots by subtracting busy periods from office hours (`services/calendar_service.py`, `providers/calendar/google.py`, `api/calendar.py`, `models/calendar_connection.py`).
- **Human-in-the-loop decisioning:** approve / reject (reason required, min 10 chars) / override-AI endpoints, each writing an audit entry (`services/candidate_service.py`).
- **Audit trail:** every action logged with user, action, entity, reason, and AI reasoning (`services/audit_service.py`, `models/audit_log.py`, migration `b2b5b126546e`).
- **Dashboard aggregates:** stats, recent activity, jobs-needing-attention, pipeline funnel (`api/dashboard.py`, `services/dashboard_service.py`).
- **Provider abstraction layer:** abstract base classes for LLM, email, calendar, storage with concrete OpenAI/Gmail/Google/MinIO implementations (`providers/`), honoring the "vendor-agnostic" principle.

Frontend (`frontend/src/app/`): full page set exists and builds — login, dashboard (with pipeline chart), jobs list, job detail, multi-step job create, candidates list, candidate detail, calendar view, audit log, plus CV-upload and reject-candidate modals and a navbar. TanStack Query hooks for each domain (`hooks/`) wired to the API client (`lib/api.ts`).

Design assets: 11 UXPilot HTML mockups for all major screens (`docs/uxpilot/`).

## 3. Planned but missing
- **Entire MVP-2 "AI Agents" phase — unbuilt.** No voice outreach, no AI screening interview, no transcript storage, no interview scoring, no retry/fallback logic. Zero references to voice/Synthflow/Bland/Twilio/Retell/WhatsApp/transcript in code (grep-confirmed). The candidate pipeline in the spec runs `RECEIVED → … → OUTREACH → INTERVIEW_SCHEDULED → INTERVIEWED → FINAL_SHORTLIST → HIRED`, but the implemented `CandidateStatus` enum stops at 6 states (`received, parsed, parse_failed, scored, shortlisted, rejected`) — no outreach/interview/hired states exist.
- **MVP-3 expansion — unbuilt:** LinkedIn integration, multi-channel unified inbox, in-portal email responses, advanced reporting, multi-user roles, Outlook calendar.
- **Outbound email/announcements:** spec wants the system to *send* job announcements and candidate notifications; only inbound polling is implemented.
- **Vector DB / semantic search:** Pinecone/pgvector and embeddings were proposed but not implemented (no embeddings, no vector deps in `requirements.txt`).
- **Tests & coverage:** 80% target; `tests/` is empty.
- **Containerized deploy:** backend/frontend Dockerfiles and compose services are commented out.
- **Bug-blocked features:** candidate list/detail endpoints and email-ingest candidate creation will fail until the `Candidate` model field mismatches are reconciled (see Snapshot/§2).

## 4. The AI gap
- **What the AI actually is:** three single-shot OpenAI `gpt-4o-mini` prompt calls — CV parsing, candidate scoring, and email→job classification. JSON-in/JSON-out, temperature ~0.3, no streaming, no tool-calling, no function-calling, no memory, no multi-step reasoning. The "LLMProvider" base is a thin completion wrapper.
- **No RAG, no retrieval, no embeddings, no vector store.** Scoring is done by stuffing the full parsed CV + criteria into one prompt; there is no knowledge base of company policy, prior hires, or role benchmarks to ground decisions.
- **No agentic behavior.** Despite the "Restless Workforce" framing, there is no planner/executor agent, no autonomous task decomposition, no self-correcting loop. The only autonomy is a fixed-interval background poller that chains parse→score.
- **Robustness gaps:** JSON parsing is hand-rolled with markdown-fence stripping (brittle); no retry/backoff on malformed LLM output; CV text truncated at 8k chars (long CVs lose data); classification silently defaults to "first active job" on any uncertainty, which can mis-route applications.
- **Model choice drift:** spec proposed Google Gemini; code ships OpenAI. Provider is swappable in principle (abstract base) but only one concrete LLM impl exists.
- **Net:** AI is genuinely *central to the value proposition* (screening/scoring is the product), but the implementation is shallow prompt engineering, not an AI-native architecture.

## 5. Missing pieces to make it sellable
1. **Fix the integration bugs and run it end-to-end** — reconcile `Candidate` model vs. service/API field names; prove email-intake→parse→score→shortlist works against a real inbox. This is gate-zero.
2. **Add a test suite + CI** — currently zero; buyers of an automated-decision HR tool need confidence and an audit-defensible QA story.
3. **Ship the voice/interview agents (MVP-2)** — the differentiating "does the recruiter's job" capability is entirely absent; without it this is a CV-scoring dashboard, not an autonomous recruiter.
4. **Ground the scoring (RAG)** — index job descriptions, company hiring rubrics, and past decisions so scores are explainable *and* consistent; add bias/fairness guardrails and an explainability export for compliance.
5. **Deploy story** — containerize, add env/secrets management (current `jwt_secret_key` and MinIO creds default to placeholders), multi-tenant data isolation, GDPR data-retention/erase controls (referenced as a principle, not implemented).
6. **Outbound comms + candidate experience** — confirmation/rejection emails, scheduling links.
7. **Reporting & ROI dashboard** — time-to-shortlist, CVs-processed, hours-saved metrics to justify the purchase.

## 6. Native-AI + Autonomous-FTE upgrade
This is one of the **most natural autonomous-FTE products** in the portfolio: an **AI Recruiting Screener / HR Intake Officer** that works 24/7. The foundation (always-on email poller, auto-parse, auto-score, audit log, human approval) is already the right shape.

Agent roles to build out:
- **Intake Officer (live, partial):** monitors the careers inbox around the clock, parses every CV, classifies to the right req, scores against criteria, drafts an explainable shortlist. *Replaces ~the manual triage/screening load of a junior recruiter — easily 10–20 hrs/week on high-volume reqs.*
- **Outreach Coordinator (roadmap):** voice + email/WhatsApp outreach to shortlisted candidates, offers interview slots from the live calendar, retries 3×, falls back to text. *Replaces scheduling coordination, ~5–8 hrs/week.*
- **Screening Interviewer (roadmap):** 5-minute adaptive voice screen, transcribes, scores communication/technical/fit, recommends for human interview. *Replaces first-round phone screens, ~minutes-per-candidate that don't scale.*
- **Compliance/Audit Clerk (live):** logs who/what/when/why and the AI's reasoning for every action — already implemented and a genuine selling point for regulated hiring.

Guardrails / human-in-the-loop (already designed in): AI **recommends**, human **approves**, system **executes**; rejections require a written reason; AI overrides are explicitly logged; every decision carries reasoning + confidence. To productionize, add: bias monitoring on score distributions, a confidence threshold below which items are force-routed to a human, PII minimization, and candidate consent capture for voice/recording.

## 7. Showcase angle (for the portal)
- **Headline:** "Your inbox full of CVs, screened and ranked by morning — every morning, no human required."
- **Stat-benefit bullets:**
  - Auto-ingests applications from the careers inbox 24/7 and parses every PDF CV into structured data — no manual data entry.
  - Scores each candidate 0–100 against your weighted criteria with a written reason per criterion — explainable, auditable shortlists.
  - Every AI decision is logged (who/what/when/why) and a human approves before anyone is contacted — control without the grunt work.
- **Demo hook:** Forward three CVs to the watched inbox, refresh the dashboard, and watch candidates appear pre-parsed, ranked, and annotated with AI reasoning — then approve the top one in one click. *(Note: requires fixing the candidate-endpoint field bug first; until then demo via the static UXPilot screens / component demo page.)*

## 8. Verdict
- **AI-native score:** 3/5 — AI is core to the value (parsing + explainable scoring + classification) and runs autonomously via the poller, but it's shallow single-shot prompting with no RAG, agents, or grounding.
- **FTE-fit score:** 4/5 — textbook autonomous-FTE concept (an AI recruiter that screens 24/7 with human approval); the always-on intake-and-score loop already embodies it. Held back from 5 because the headline "does the interview" agents (voice/screening) are unbuilt.
- **Maturity:** Prototype (early MVP backend with runtime-blocking integration bugs, no tests, no deploy; full frontend page set and design system in place).
- **Recommendation:** **Incubate** — strongest natural autonomous-FTE fit of the HR products and a large, real buyer pain, but it needs (1) the model/service bug fixes to even run end-to-end, (2) a test suite, and (3) the MVP-2 voice/interview agents before it can be a credible flagship. Fast path from Incubate → Flagship for the recruiting vertical once stabilized.
