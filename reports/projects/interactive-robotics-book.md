# Interactive Robotics Book — an AI-native interactive textbook for Physical AI & Humanoid Robotics (clean reboot, governed by Constitution 2.0)

> Honesty note up front: `interactive-robotics-book` (v2.0) is a **clean-reboot repository** that today contains **only scaffolding + specifications** — every source directory holds a `README.md` placeholder and nothing else. The working features described below (3 live chapters, Gemini chat, Urdu translation, search, live URLs) were shipped by the **v1.0 predecessor** documented in `/home/anjum/dev/book_ai` (repo `AI_Robotics_Bppl`). They have **not** been migrated into or rebuilt in the v2.0 repo. This report keeps "built today in the showcased repo" strictly separate from "proven in v1.0" and from "roadmap."

## 1. Snapshot
- **One-liner:** An AI-native, GitHub-published interactive textbook platform for Physical AI / Humanoid Robotics, with an in-page AI tutor and Urdu translation — currently a spec-and-scaffold reboot of a partially-shipped v1.0.
- **Category:** EdTech (interactive technical publishing / online textbook)
- **Target buyer:** Robotics/AI educators and bootcamps; universities and corporate L&D building Physical-AI curricula; self-learners (B2C). Secondary: technical authors/publishers wanting an AI-assisted authoring + tutoring stack.
- **Tech stack:** Docusaurus 3.6 + React 18 + TypeScript/MDX (frontend, GitHub Pages); Next.js 16 + Prisma 6 + better-auth on Vercel Edge (backend); Neon serverless PostgreSQL; **Google Gemini 2.0 Flash** for chat + translation (note: not Claude/Anthropic); local full-text search (`@easyops-cn/docusaurus-search-local`); Playwright/Jest/Lighthouse CI; GitHub Actions. Governed by SpecKit Plus + "Constitution 2.0."
- **Maturity:** **Prototype (scaffold + specs).** Justification: the v2.0 repo has 5 git commits, all foundation/spec work (`git log`: foundation → SpecKit → Phase 1 files → Phase 2 specs). Backend = one file (`backend/lib/shared/types.ts`, 51 lines of interfaces); no API routes, no Prisma schema, no AI code. Frontend = Docusaurus config + sidebars + `package.json`; `frontend/docs/` holds only `README.md` (zero chapters), `frontend/src/components/` only a `README.md`. There are 6 detailed feature specs (993 lines total) but no implementations. The v1.0 predecessor reached "Alpha / ~20% complete" (per book_ai post-mortem); v2.0 is pre-MVP foundation.
- **Live URL / demo:** v2.0 GitHub Pages target configured (`https://shehzadanjum.github.io/interactive-robotics-book/`) but **would not yet render a book** — no content and `onBrokenLinks: 'throw'` with a missing `/intro`. The **working v1.0 demo** lived at `https://shehzadanjum.github.io/AI_Robotics_Bppl/` (frontend) and `https://airobobookmagic.vercel.app/` (backend) per book_ai docs.
- **Path:** /home/anjum/dev/interactive-robotics-book (+ analysis in /home/anjum/dev/book_ai)

## 2. What's built today (verified)
**In the showcased v2.0 repo (`interactive-robotics-book`) — file-evidenced:**
- **Repository scaffolding only.** Every functional directory contains a placeholder `README.md` and no code: `frontend/docs/`, `frontend/src/`, `frontend/src/components/`, `frontend/hooks`, `frontend/lib`, `frontend/theme`, `frontend/static/img`, `frontend/tests/{e2e,unit}`, `backend/app/api`, `backend/prisma`, `backend/prisma/migrations`, `backend/tests`.
- **Frontend config (no content):** `frontend/docusaurus.config.ts` (Mermaid + local search wired), `sidebars.ts`, `package.json`, `tsconfig.json`. No `.md`/`.mdx` chapters exist. No `src/css/custom.css` referenced by config. Search plugin is configured but has nothing to index.
- **Backend config (no logic):** `backend/package.json` (Next 16, Prisma, better-auth, zod), `next.config.mjs`, `tsconfig.json`, and a single `lib/shared/types.ts` defining `User / Chapter / Bookmark / Progress / ChatMessage / ChatResponse` interfaces. **No API routes, no Prisma schema/migrations, no Gemini integration, no auth wiring, no seed.**
- **6 feature specifications (the real asset here):** `001-book-platform`, `002-ai-chat-widget`, `003-translation`, `004-search`, `005-authentication`, `006-user-features` — each with `spec.md`, `spec.backup.md`, and a requirements checklist; 993 spec-lines total, with structured clarifications, FRs, and measurable success criteria (e.g., the chat spec defines streaming SSE, text-selection context, per-user rate limits 5/min·100/hr, RAG-later plan).
- **Governance + process tooling:** `Constitution 2.0` (12 enforced principles), SpecKit Plus templates/scripts under `.specify/`, PHRs (Prompt History Records) for every spec, planning docs, and Gemini slash-command definitions under `.gemini/commands/`.
- **Agents/skills are stubs:** `agents/{content-validator, deployment-agent, beta-tester-agent, professor-agent, editor-agent}` and `skills/{chapter-writer-skill, code-example-generator, source-validator}` exist as **README-only directories**; README explicitly labels them "To be implemented" / "Future."

**Proven in v1.0 (separate `book_ai`/`AI_Robotics_Bppl` repo — NOT in the showcased repo, per book_ai post-mortem):**
- 3 chapters actually written and deployed (~19,953 words; ~20% of a 12–15 chapter goal), with 100% three-source validation and <2s page loads.
- Working **Gemini 2.0 Flash chat widget** (streaming, session persistence) and **Urdu translation** (RTL) — both genuinely operational, not stubbed.
- Working local full-text search (626KB index, offline), Neon Postgres tables, and 5 GitHub Actions auto-deploy workflows. Authentication was "95% complete" but **never connected to the UI / not deployed**.

## 3. Planned but missing
All of the following are specified in v2.0 but **not built in the v2.0 repo**:
- The 3 chapters themselves (migration from v1.0 not done) and chapters 4–15.
- AI chat widget (spec `002`): floating Gemini assistant, SSE streaming, text-selection "Ask AI"/"Translate" actions, local-storage history, rate limiting, planned RAG via per-chapter embeddings.
- Urdu translation system (spec `003`), search UI/index (spec `004`), email + Google OAuth authentication (spec `005`), and user features — progress tracking, bookmarks, personalization (spec `006`).
- The agent/skill automation (content-validator, deployment, beta-tester, professor, editor) — all stubs.
- Working build/deploy of the v2.0 site (currently would fail to produce a populated book).

## 4. The AI gap
- **AI is central to the product vision, but absent from the showcased repo.** There is zero AI code in `interactive-robotics-book` today — only an interface type (`ChatResponse`) and a richly written chat spec.
- **What AI genuinely did work (v1.0):** Gemini-powered Q&A chat and Urdu translation. These are real, useful AI features — but they are **assistive**, not autonomous, and they live in a different repo.
- **Authoring was not autonomous.** Per book_ai post-mortem, each chapter took **6–8 hours with heavy human involvement**: human research + outline (1.5–3h), AI-assisted drafting (3–4h), human accuracy review (1–2h), manual corrections/validation (0.5–1h). The "professor/editor multi-persona" pipeline was **never formally implemented**; it remained a manual workflow.
- **No RAG, no retrieval grounding.** Chat used current-chapter context only; RAG was deliberately deferred ("RAG after content"). With only 3 chapters, accuracy grounding was thin.
- **The provider is Gemini, not Claude.** Any Aevum framing should note the model swap (and the opportunity to upgrade authoring/tutoring onto a stronger agentic stack).
- **Net:** The product is "AI-assisted publishing" in concept and a working-but-modest AI tutor in v1.0 — it is **not** an autonomous-authoring system today.

## 5. Missing pieces to make it sellable
1. **Ship something live.** Migrate the 3 v1.0 chapters into the v2.0 repo and get a working public demo (current config can't render a book). Without a live artifact, this is a spec, not a product.
2. **Reconnect the AI tutor end-to-end** in v2.0 (Gemini chat + translation), the one demonstrably valuable, sticky feature.
3. **Content depth.** A 3-chapter (20%) textbook is a teaser, not a sellable curriculum; need a credible path to 12–15 validated chapters.
4. **Author productivity tooling.** The 6–8h/chapter manual pipeline is the actual cost center — turning that into a fast, governed agent loop is the value story.
5. **Accuracy/trust layer.** RAG grounding + the existing three-source-validation rule wired into the pipeline (so the AI tutor cites chapter sources; so generated content is auditable).
6. **User validation.** v1.0 shipped 3 chapters with **zero real-student testing** — a noted risk. A beta-feedback loop is needed before claiming pedagogical efficacy.
7. **Commercial basics:** auth actually deployed (it never was), accounts/progress, a license (README still says "Choose License"), and a packaging/pricing model (per-seat, per-cohort, white-label for educators).

## 6. Native-AI + Autonomous-FTE upgrade
- **AI Textbook-Author agent (the headline FTE):** An autonomous, governed authoring loop that turns a syllabus item into a validated interactive chapter — researches sources (enforcing the three-source rule), drafts MDX with runnable code + Mermaid diagrams, self-reviews via a professor/editor critic pass, and opens a PR through the existing deployment agent. This directly automates the v1.0 manual pipeline.
  - **"Replaces ~N hours" math (grounded in book_ai metrics):** v1.0 measured **6–8 hours of human-supervised work per chapter**. A 15-chapter book ≈ **90–120 author-hours**; the post-mortem also logged **26–38 hours lost to rework/context-loss** that an agentic loop with persistent state avoids. Pitch: *"one autonomous curriculum-author FTE drafts a validated chapter in hours instead of a day, replacing ~90–120 hours of authoring per textbook and recovering the ~30 hours v1.0 burned on rework."*
- **AI Tutor / TA agent (24/7):** The Gemini-style in-page tutor, upgraded with RAG grounding over the full book, citation of source chapters, and per-learner progress awareness — a round-the-clock teaching assistant. Pitch: *"replaces the front-line Q&A load of a course TA across an unlimited learner base, 24/7."*
- **Guardrails/accuracy (non-negotiable for EdTech):** RAG retrieval with inline citations; the three-source-validation gate enforced in CI (Constitution 2.0 already mandates it, just never automated); human-in-the-loop PR approval for new chapters; rate limiting + cost caps (already specified); and a "redirect off-topic" guard (already in the chat spec).
- **Reality check for the showcase:** the agent/skill scaffolding exists in name only — this is a **build**, leveraging proven v1.0 AI features as the foundation, not a wrap-up.

## 7. Showcase angle (for the portal)
- **Headline:** "The textbook that writes and teaches itself — autonomous AI authors the chapters, a 24/7 AI tutor answers every learner."
- **Stat-benefit bullets:**
  - **~90–120 author-hours per textbook → hours:** an autonomous author-agent drafts validated, source-cited chapters, collapsing the v1.0-measured 6–8h/chapter manual pipeline.
  - **24/7 AI tutor, proven in v1.0:** Gemini-powered chat + instant Urdu translation already worked live — context-aware Q&A and one-click "explain this" on any highlighted passage.
  - **Governed for trust:** three-source validation + RAG citations baked into a constitutional, auto-deploying CI pipeline — accuracy you can audit.
- **Demo hook:** Highlight a paragraph on "actuator kinematics" → AI tutor explains it inline and offers an Urdu translation; then click "Generate next chapter" and watch the author-agent open a validated, cited PR. (Honest caveat for internal use: tutor/translation are reusable from v1.0; the author-agent is the net-new build.)

## 8. Verdict
- **AI-native score:** 3/5 — AI is core to the concept and v1.0 shipped genuinely working AI tutoring/translation, but the showcased repo has zero AI code today, no RAG, and authoring was never autonomous.
- **FTE-fit score:** 2/5 — the "autonomous curriculum-author + 24/7 TA" mapping is excellent and quantifiable, but as-built it's a semi-manual, human-heavy pipeline plus stubbed agents; the FTE is a roadmap, not a deployed worker.
- **Maturity:** Prototype (scaffold + 6 specs + governance); v1.0 predecessor reached working Alpha (~20% of scope) but is unmigrated.
- **Recommendation:** **Incubate.** Strong specs, real governance discipline, and a proven-but-separate v1.0 give it a credible foundation — but nothing is shipped in the showcased repo. It is not a flagship until (a) the 3 chapters + AI tutor are live in v2.0 and (b) at least a thin author-agent demo exists. Highest-leverage next step: migrate v1.0's working tutor + chapters, then build the author-agent as the differentiating FTE.
