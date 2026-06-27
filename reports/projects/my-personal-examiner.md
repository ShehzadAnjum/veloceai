# My Personal Examiner — An AI Economics examiner-tutor that marks A-Level answers to A* standard, 24/7

## 1. Snapshot
- **One-liner:** A FastAPI + PostgreSQL platform that uses LLMs to teach, examine, mark, and coach Cambridge A-Level Economics students at "PhD-examiner" strictness — modelled as six AI teaching roles (Teacher, Coach, Examiner, Marker, Reviewer, Planner).
- **Category:** EdTech (AI tutoring + automated examination/marking)
- **Target buyer:** A-Level students (B2C) sitting Cambridge International exams; secondary buyers are tutoring centres / schools / exam-prep tutors who would otherwise pay for human marking and 1:1 tuition.
- **Tech stack:** Backend — FastAPI 0.124, SQLModel, PostgreSQL 16 (Neon) + pgvector, Alembic (29 migrations), Celery scaffolding. AI — Anthropic Claude Sonnet 4 (`claude-sonnet-4-20250514`), OpenAI GPT-4 Turbo, Gemini 2.0 Flash, with a fallback orchestrator + circuit breaker; Gemini/OpenAI embeddings for RAG. Frontend — Next.js 15 / React 19, TypeScript, Tailwind, shadcn/Radix, TanStack Query, better-auth, KaTeX/Mermaid/markdown. Deploy — Oracle Cloud VM via PM2 + nginx (also stale Vercel config).
- **Maturity:** **Prototype** (advanced, content-starved). Justification: ~21 routed FastAPI services, six real LLM-agent prompt suites, a large Next.js app (admin "Explanation Studio" alone is 3,853 lines), pgvector semantic search, and a staged cloud deploy all exist — far beyond "idea." But it is **not demonstrable end-to-end**: the only subject is Economics 9708 with ~55–166 syllabus points, 10 sample questions and **zero mark schemes seeded**; the resource/question banks are nearly empty; marking was "never validated"; key LLM service paths crash on a tuple-unpack bug; exam-taking/results/planner UIs are missing; and the test suite reports **262 pass / 113 fail / 36% coverage**. That is a prototype, not an MVP.
- **Live URL / demo:** No confirmed public demo. A deploy was staged to the Oracle VM `http://161.118.195.63:3080` (latest commit: "feat: deploy My Personal Examiner to Oracle Cloud"), but the deploy script warns the backend `.env` must be created manually and there is no evidence it is serving. README's Vercel link is a literal placeholder. No live URL recorded.
- **Path:** /home/anjum/dev/my_personal_examiner

## 2. What's built today (verified)
- **Genuine multi-provider LLM layer** — `backend/src/ai_integration/` has real SDK clients for Anthropic, OpenAI and Gemini plus `llm_fallback.py` (`LLMFallbackOrchestrator`: exponential backoff, 5-failure circuit breaker, Anthropic→OpenAI→Gemini chain or pinned single-provider mode). Not stubs — real `messages.create` / completion calls.
- **Six AI "teaching role" prompt suites** — `backend/src/ai_integration/prompt_templates/{teacher,coach,marker,reviewer,planner}_prompts.py`. The Marker system prompt (verified, `marker_prompts.py:23`) is a detailed "PhD-level examiner, zero tolerance" prompt with a strict JSON output schema (AO1/AO2/AO3 scores, error categorisation, point-by-point quotes, confidence, feedback).
- **Real LLM marking against a mark scheme** — `backend/src/services/marking_service.py` fetches the question + Cambridge mark scheme, calls the LLM at temp 0.1, parses AO1/AO2/AO3 marks and errors, then applies a 6-signal heuristic confidence score (`src/algorithms/confidence_scoring.py`) and flags <70% for human review. This is LLM-as-examiner, not string matching. Cambridge grade boundaries (A*≥90…U<40) and overall-feedback assembly are rule-based.
- **A working DB-free practice marker** — `backend/src/routes/marking.py:305` `/check-practice-answer` calls Gemini (→OpenAI fallback) directly with strikethrough-correction formatting; this path works (unpacks correctly).
- **Real AI teaching + RAG** — `backend/src/services/teaching_service.py` generates 9-component structured explanations (LLM @ temp 0.3, up to 8000 tokens), injects retrieved resource text as context, validates/repairs the JSON, and persists `GeneratedExplanation` with provider/model/token metadata.
- **Multi-turn Socratic coaching loop** — `backend/src/services/coaching_service.py` persists a `CoachingSession` transcript and replays history (temp 0.7). Real conversation design.
- **Embeddings + pgvector semantic search** — `backend/src/services/embedding_service.py` + `routes/embeddings.py`/`search.py`: real cosine-similarity SQL over `SyllabusPoint.embedding`, subject-scoped content alignment, batch embedding jobs.
- **Rule-based exam assembly** — `exam_generation_service.py` builds papers from the existing question bank via random/balanced/syllabus-coverage selection (no LLM); duration heuristic 2 min/mark.
- **Production-grade domain model** — 29 SQLModel entities: full curriculum hierarchy (`AcademicLevel → Subject → Syllabus → SyllabusPoint` with self-referential parent/child), examiner metadata (`CommandWord`, `AssessmentObjective`, `PaperTemplate`, `MarkScheme`), assessment (`Question`, `Exam`, `Attempt`, `AttemptedQuestion`), AI-tutoring (`CoachingSession`, `StudyPlan` SM-2, `ImprovementPlan`), resource bank, per-student encrypted LLM keys, activity/login audit. 29 Alembic migrations.
- **Substantial Next.js frontend wired to the API** — dashboard, teaching/learning, coaching chat, resource bank, and a deep admin panel (academic setup, Explanation Studio, users, questions, resources) under `frontend/app/`; typed API clients in `frontend/lib/api/`; better-auth (bcrypt/JWT + Google OAuth).
- **Past-paper extraction pipeline** — `backend/src/question_extractors/` (Cambridge parser, mark-scheme extractor, regex patterns) parses real past-paper PDFs; 16 PDFs on disk.
- **Real test suite (partial)** — 568 test functions across 37 unit/integration files (e2e/accuracy are empty scaffolds); coverage report shows 36%.

## 3. Planned but missing
- **Content.** The system is scaffolding starved of data: only Economics 9708; ~55–166 syllabus points and **10 sample questions**, **0 mark schemes**, resource/explanation bank "empty for most topics," `backend/resources/` config JSON missing. `tasks/todo.md` (the newest, most candid doc, 2026-03-09): *"Question bank nearly empty… no Cambridge extraction done."*
- **End-to-end student cycle.** Exam-taking UI, results/feedback UI, and study-planner UI are not built, so the intended learn→coach→exam→mark→review→plan loop cannot be demonstrated.
- **Marking validation.** Per `tasks/todo.md`: *"Marking not validated… never tested."* The ≥85%-accuracy target in `specs/phase-3-ai-teaching-roles/spec.md` is unproven.
- **Syllabus-agent pipeline integration.** A separate `src/syllabus_agents/` package (real LLM question generation, syllabus extraction) exists but is *"never run end-to-end"* and is not wired into the exam/question routes.
- **Multi-subject expansion.** The schema supports any board/subject (AS/A/O/IGCSE/IB) but only Economics is seeded.
- **Test/quality gate.** 113 failing tests and 36% coverage vs the project's own 80% "constitutional" requirement.

## 4. The AI gap
The AI here is **real and architecturally central**, not bolted on — LLMs are the product (the examiner, the tutor, the coach), backed by genuine prompts, RAG, and a multi-provider fallback. That is the strength. The gaps are execution and integrity, not vision:
- **Two LLM service paths crash as written.** `generate_with_fallback` returns a 3-tuple `(completion, provider, model_name)`, but `marking_service.py:149`, `coaching_service.py:141/318`, `review_service.py` and `resource_service.py` unpack only 2 values → `ValueError: too many values to unpack` before the LLM output is used. So the DB-backed marking and coaching flows are wired to real LLMs but would throw at runtime. (Teaching and the `/check-practice-answer` path unpack correctly and work.)
- **Model mislabelling.** Every Anthropic docstring says "Claude Sonnet 4.5," but the configured ID `claude-sonnet-4-20250514` is Claude Sonnet 4. A showcase should fix the label (or upgrade the model) before claiming "4.5."
- **Per-student keys stored but never used.** `llm_key_service.py` + `StudentLLMConfig` encrypt keys with Fernet, but every generation call constructs clients with no `api_key` arg — the system always uses server env keys. The "bring your own key" capability is half-wired.
- **No marking accuracy evidence.** "PhD-level A* marking" is asserted via prompt, not validated against real Cambridge mark schemes/examiner scores; the accuracy test suite is an empty scaffold.

## 5. Missing pieces to make it sellable
1. **Load real content** — full Economics 9708 syllabus, a real past-paper question bank, and seeded Cambridge mark schemes (the extraction pipeline exists; run it). Without questions and mark schemes there is nothing to examine.
2. **Fix the 3-tuple unpack bug** across marking/coaching/review/resource services so the flagship marking and coaching flows actually run.
3. **Build the missing UIs** — exam-taking, results/feedback, and study planner — to close the demonstrable student loop.
4. **Validate marking accuracy** against real examiner-marked answers and publish the number (target ≥85%); this is the single most sellable proof point for "AI examiner."
5. **Wire per-student keys** (or drop the feature) and correct the Sonnet 4/4.5 labelling.
6. **Security hygiene** — live Neon Postgres credentials and a Vercel OIDC token are committed in `frontend/.env.local`, `frontend/.env.vercel`, and root `.env.local`; rotate and remove before any showcase.
7. **Stabilise the test gate** — clear the 113 failures and lift coverage toward the stated 80%.
8. **Confirm and pin a live demo URL** (the Oracle VM deploy is staged, not verified).

## 6. Native-AI + Autonomous-FTE upgrade
Position this as an **Autonomous A-Level Examiner & Tutor FTE** for Cambridge Economics, working 24/7:
- **AI Examiner/Marker (flagship):** ingests a student answer + Cambridge mark scheme, returns AO1/AO2/AO3 breakdown, point-by-point evidence quotes, error categorisation, A*–U grade, and improvement feedback at temp 0.1 for consistency — with a confidence score that **auto-escalates low-certainty marks (<70%) to a human review queue** (the integrity guardrail is already modelled in `AttemptedQuestion.needs_review`). Replaces the bulk of a human marker's turnaround: a marker spends ~8–15 min per structured Economics answer; this returns it in seconds, any hour.
- **AI Tutor/Teacher:** generates PhD-level structured explanations with diagrams (Mermaid) and math (KaTeX), grounded in retrieved syllabus/resource context (RAG) to curb hallucination.
- **AI Coach:** multi-turn Socratic dialogue that diagnoses misconceptions and adapts — the part students normally pay £30–50/hr of tutor time for.
- **AI Planner:** SM-2 spaced-repetition study scheduling off weak AOs.
- **"Replaces ~N hours":** a credible framing is *"one Autonomous Examiner FTE absorbs the marking + first-line tutoring of roughly 1–2 human tutors per cohort"* — i.e. unlimited 24/7 marking and explanation at near-zero marginal cost, with humans kept in the loop only on flagged low-confidence marks.
- **Guardrails / academic integrity:** confidence-gated human review, mark-scheme-anchored grading (not free-form opinion), RAG-grounded explanations, full audit trail (`ActivityLog`, per-attempt feedback persistence), and provider fallback for uptime. These need the marking-accuracy validation (Section 5) to be defensible.

## 7. Showcase angle (for the portal)
- **Headline:** "The A* examiner who never sleeps — instant, mark-scheme-strict grading and tutoring for A-Level Economics."
- **Stat-benefit bullets:**
  - **Six AI teaching roles, one engine** — Teacher, Coach, Examiner, Marker, Reviewer, Planner, all built on real Claude/GPT/Gemini with automatic failover.
  - **Examiner-grade marking** — AO1/AO2/AO3 breakdown, error categorisation, and A*–U grading in seconds, with <70%-confidence answers auto-routed to a human (no silent guessing).
  - **Grounded, not hallucinated** — explanations and marks are anchored to the Cambridge syllabus and mark scheme via pgvector RAG.
- **Demo hook:** Paste a weak Economics essay answer; watch the AI examiner mark it point-by-point against the mark scheme, award a grade, flag its own uncertainty, and hand back a tutor-style "here's how to reach A*" rewrite — live, in seconds. (Use the working `/check-practice-answer` path for the demo while the DB marking path is fixed.)

## 8. Verdict
- **AI-native score:** 4/5 — LLMs are the core product (examiner/tutor/coach), with real prompts, RAG, embeddings and multi-provider orchestration; docked one point because flagship paths crash on a known bug and marking accuracy is unproven.
- **FTE-fit score:** 4/5 — the "autonomous examiner + tutor working 24/7, humans only on flagged marks" framing is a near-perfect fit for the Autonomous-FTE story; held back only by content-starvation and unvalidated accuracy.
- **Maturity:** Prototype (advanced but content-starved; not demonstrable end-to-end).
- **Recommendation:** **Secondary.** Strong concept and unusually deep architecture for the EdTech line, but it needs content loading, a handful of bug fixes, marking validation, and the missing exam UIs before it can headline. With ~2–4 weeks of focused work (run the extraction pipeline, fix the unpack bug, validate marking) it is a credible Flagship candidate for the AI-examiner narrative.
- **Relationship note:** The sibling `/home/anjum/dev/A-Level-Learning` (Prisma) is a separate, cleaner rebuild of the same A-Level EdTech idea, covered by another analyst. `my_personal_examiner` is the older, far richer-but-messier FastAPI original — the deeper examiner/marking engine of the two.
