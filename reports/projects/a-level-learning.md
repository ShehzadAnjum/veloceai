# Cambridge AI Professor — A-Level exam-prep platform that is "AI" in name only (today)

## 1. Snapshot
- **One-liner:** A Cambridge A-Level study tool that ingests real past papers/mark schemes and runs an "A\*-Workflow" learning loop (Assign → Test → Diagnose → Remediate → Model) — currently mid-rebuild and with zero real AI.
- **Category:** EdTech (exam prep, Cambridge International A-Levels).
- **Target buyer:** A-Level students (B2C) and tutoring centres / schools prepping Cambridge candidates (B2B2C). Subjects covered by assets: Maths 9709, Economics 9708, Accounting 9706, English GP 8021.
- **Tech stack:** Two layers. (1) Legacy Python CLI: SQLAlchemy + SQLite, Alembic, `pdfplumber`, `requests`/BeautifulSoup scraper. (2) Rebuilt webapp: Next.js 14 (App Router, React 19, TS strict, Tailwind 4, shadcn/ui, Zustand) + FastAPI + Prisma (Python client) on Neon PostgreSQL. Playwright/Vitest/pytest scaffolding. SpecKit Plus (`/sp.*`) spec-driven workflow.
- **Maturity:** **Prototype (rebuild-in-progress).** Despite docs claiming "98% complete / production-ready," the core learning-loop endpoints are unimplemented stubs (see §2). Justification: the web app cannot complete a single end-to-end study session.
- **Live URL / demo:** None. Local only (frontend :3000, backend :8000). No deployment despite a configured Neon URL in `webapp/backend/.env`.
- **Path:** /home/anjum/dev/A-Level-Learning

## 2. What's built today (verified)
- **Real domain assets (the genuine value).** `resource_bank/` holds actual Cambridge materials — Past_Paper / Mark_Scheme / Examiner_Report folders for 4 subjects across 2021–2025, plus syllabuses and textbooks. `download_past_papers.py` is a working scraper (savemyexams/Cambridge sources). This is a real, hard-to-assemble content moat.
- **Clean web scaffolding.** Next.js front end has a full staged UI: `TopicSelector`, `LearningPackDisplay`, `ExamInterface`, `ResultsDashboard`, `ProgressStepper`, error boundaries, loading states, lazy loading (`webapp/frontend/src/app/(dashboard)/loops/page.tsx`). Backend has a layered routes→services→Prisma architecture and a complete 8-model + 3-join Prisma schema (`webapp/backend/prisma/schema.prisma`: Student, Resource, SyllabusPoint, LearningPack, Question, MockExam, ExamAttempt, AttemptedQuestion).
- **Working CRUD-ish endpoints.** Resources/Students list/get/create exist. DB connection tests pass per `FIXES_APPLIED.md`.
- **The core feature does NOT work.** In `webapp/backend/app/services/loop_service.py`, `generate_test()` and `submit_test()` both `raise ResourceUnavailableError("...being migrated to Prisma")`. So: a student can start a loop and create a learning pack, but **cannot generate a test, take it, or get results** through the web app. The whole "A\*-Workflow" headline flow is non-functional.
- **The legacy CLI "works" but is fake.** `src/a_star_workflow_orchestrator/orchestrator.py` "grades" answers with `score = random.uniform(0, max_marks) if len(user_answer) > 5 else 0` and "diagnoses" weakness as "score < 50% → review topic." Explicitly self-labeled "dummy logic" / "Simulated." Content/pack generation is "placeholder logic."
- **Heavy status-doc inflation.** 12+ markdown reports (FINAL_STATUS, REBUILD_SUMMARY, PRISMA_SETUP_COMPLETE, ENV_*, FIXES_APPLIED…) assert "production-ready" / "best practices" / "98%". Code does not support these claims. Version churn is visible (docs cite Prisma 6.0.0, root `package.json` pins Prisma 7.1.0, a stray file literally named `7.1.0` sits in `webapp/backend/`).

## 3. Planned but missing
- Completing the Prisma migration of `generate_test` / `submit_test` (the actual product).
- PDF → individual-question extraction at quality (`pdf_qa_extractor.py` / `learning_pack_generator.py` are prototypes/placeholders).
- Syllabus points are auto-created as placeholders ("subject: Math" hardcoded default) instead of a real syllabus taxonomy.
- Auth/JWT (noted as "future"), multi-user, persistence of loop state (currently an in-memory dict, lost on restart).
- Deployment (Vercel + Neon), CI/CD, real test coverage (infra exists, assertions thin).

## 4. The AI gap
- **There is no AI in this product.** No `openai`, `anthropic`, `google-generativeai`/`genai`, or `langchain` anywhere in `requirements.txt` or frontend deps. `GEMINI.md` is a **Gemini-CLI developer-agent rules file** (SpecKit/PHR/ADR tooling used to build the repo), not a runtime model integration.
- Everything branded "AI Professor," "Diagnose," "Remediate," "Model answers," "personal tutor" is either a **random-number simulation** (CLI) or an **unimplemented stub** (web app). The UI literally says "Your personal A-Level tutor, powered by the A\*-Workflow" while no grading, tutoring, or generation model exists.
- Net: the gap between the marketing surface and the engine is the entire product. This is a content+scaffolding asset waiting for an AI brain.

## 5. Missing pieces to make it sellable
- A real **LLM auto-grader** that scores free-text answers against the actual mark schemes already in `resource_bank/` (rubric-grounded, citing mark points).
- An **LLM question/pack generator** grounded in retrieved past-paper questions + syllabus (RAG over the existing PDFs).
- Finish the loop end-to-end and persist it; add auth + per-student progress dashboards.
- Replace placeholder syllabus seeding with real Cambridge syllabus codes per subject.
- Honest status docs, a deployed demo, and at least one fully working subject vertical.

## 6. Native-AI + Autonomous-FTE upgrade
The unfair advantage here is the **mark schemes and examiner reports** — the exact rubric an examiner uses. Rebuild the loop around three always-on agents:
- **AI Auto-Examiner (24/7 grader):** ingests a student's free-text answer + the matching mark scheme/examiner report, awards marks point-by-point, and writes examiner-style feedback. *Replaces ~3–5 hours/week of a human tutor's marking per student, instantly and at 3am.* Guardrail: every mark must cite a specific mark-scheme point; show confidence and "needs human review" flags for borderline scripts.
- **Adaptive-Learning Tutor agent:** from graded attempts, identifies weak syllabus points and auto-assembles the next learning pack + targeted drill set (RAG over the past-paper bank). *Replaces the lesson-planning/curation hours of a tutor.* Guardrail: only surface content traceable to real resources (no hallucinated questions).
- **Model-Answer & Walkthrough agent:** generates worked, mark-scheme-aligned model answers on demand. Guardrail: grounded in retrieved mark schemes; flagged as AI-generated.
Together these turn the "A\*-Workflow" from a 5-stage simulation into a genuine autonomous-FTE tutor/marker — the cleanest "replaces N tutor-hours" story in the EdTech line.

## 7. Showcase angle (for the portal)
- **Headline:** "The A-Level examiner that never sleeps — marks against the real mark scheme, 24/7."
- 3 stat-benefit bullets:
  - Grades free-text answers against **real Cambridge mark schemes** (4 subjects, 2021–2025 already loaded) — point-by-point, examiner-style.
  - **~3–5 tutor marking-hours/week per student** offloaded to an always-on agent.
  - Closed adaptive loop: diagnose weakness → auto-build the next study pack → re-test, with zero human scheduling.
- **Demo hook:** Paste an essay answer, watch the agent award marks line-by-line citing the mark scheme, then auto-generate the student's next targeted practice set.

## 8. Verdict
- **AI-native score:** 0.5/5 (no model anywhere; "AI" is a random-number simulation and stubs — but the rubric-rich content assets make a real AI rebuild unusually viable).
- **FTE-fit score:** 1.5/5 (the auto-examiner concept is a textbook autonomous-FTE fit, but nothing autonomous exists yet).
- **Maturity:** Prototype / stalled rebuild (core loop non-functional; docs overstate to "production-ready").
- **Recommendation:** **Incubate.** Park the inflated "98% done" narrative, but do not discard — the past-paper + mark-scheme corpus is a genuine moat and the mark-scheme-grounded auto-grader is a strong AI-native story. Position as the **B2C/student-facing front end of the "A-Level EdTech" line**, paired with the sibling.
- **Relationship to `my_personal_examiner` (sibling, separate analyst):** That project is a FastAPI "PhD-level examiner" and is almost certainly the **more advanced and more AI-native** of the pair (this one ships no LLM at all). Likely play: my_personal_examiner provides the real grading/examiner brain; A-Level-Learning contributes the Cambridge content bank and the student-facing Next.js UI/loop. Treat A-Level-Learning as the secondary/UI+content half, not the flagship.

---
SLUG=a-level-learning | NAME=Cambridge AI Professor | CATEGORY=EdTech | MATURITY=Prototype (stalled rebuild) | AI=0.5/5 | FTE=1.5/5 | VERDICT=Incubate | HOOK=A-Level marking agent grounded in real Cambridge mark schemes
