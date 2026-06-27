# My Skills Collection — the capability library that gives Aevum's FTEs their hands

## 1. Snapshot
- **One-liner:** A 71-skill library of battle-tested Claude Code agent skills (browser automation, document I/O, email/WhatsApp comms, DB/auth patterns, cloud deploy, domain extractors) packaged as portable `SKILL.md` capability modules.
- **Category:** Platform Capability / Skill Library (not a sellable app).
- **Target buyer:** Internal / platform — consumed by every Aevum autonomous FTE, not sold standalone.
- **Tech stack:** Markdown `SKILL.md` definitions (Claude Code skill format) + ~65 bundled Python scripts, Bash helpers, Playwright MCP client, Go bridge integrations; references in `references/` subdirs. No runtime of its own — skills ride on the agent host.
- **Maturity:** **MVP** (trending toward Production for the top ~15 skills). Justification: 71 working skill modules, 26 with executable `scripts/`, 21 with `references/`, ~14k lines of SKILL.md content — many encode hard-won "lesson learned" gotchas from real projects (dated notes like "PromptForge, Feb 2026"). Held back from Production by packaging hygiene: README lists only 21 of 71 skills, and ~12 stray duplicate `.md`/`.skill` files sit loose in root mirroring directory skills. No automated quality gate runs across the catalog.
- **Live URL / demo:** None. Distributed via `git clone` + copy to `~/.claude/skills/` (GitHub: ShehzadAnjum/my_skills per README).
- **Path:** /home/anjum/dev/my_skills

## 2. What's built today (verified)
71 `SKILL.md` modules verified on disk (`find ... -name SKILL.md | wc -l` = 71), plus loose `.md`/`.skill` variants. Catalog by category (counts approximate, grouped from folder scan):

- **Document processing & generation (~7):** `docx`, `pdf`, `pptx`, `xlsx`, `graph-creation`, `theme-factory`, `doc-coauthoring`. Office-format read/write/edit with formulas, tracked changes, layouts.
- **Comms & external API integrations (~8):** `gmail-oauth-email`, `whatsapp-integration`, `google-places-api`, `gemini-llm-integration`, `anthropic-api-patterns`, `chatkit-integration`, `voice-chat-bilingual`, `browser-notifications`. Real reach-out channels for an FTE.
- **Web & browser automation (~3):** `browsing-with-playwright`, `webapp-testing`, `fetch-library-docs`. Lets an agent drive a real browser and scrape/test.
- **Backend / data / DB patterns (~9):** `neon-sqlmodel`, `sqlmodel-database-schema-design`, `alembic-migration-creation`, `postgresql-client`, `multi-tenant-security`, `multi-tenant-query-pattern`, `mcp-crud-design`, `pydantic-schema-validation`, `fastapi-route-implementation`.
- **Auth & security (~3):** `better-auth-setup`, `better-auth-jwt`, `bcrypt-password-hashing`.
- **DevOps / cloud / deploy (~11):** `azure-aks-deployment`, `oracle-cloud-deploy`, `vercel-deployment`, `vercel-fastapi-deployment`, `docker-minikube`, `github-actions-cicd`, `kafka-dapr-patterns`, `cloud-native-blueprint`, `dev_ops`, `port-management`, `uv-package-management`.
- **Frontend (~3):** `react-pagination`, `shadcn-ui-components`, `tailwind-css-colors`.
- **Meta / skill-authoring & repo hygiene (~10):** `skill-creator`, `skill-creator-pro`, `skill-validator`, `context-hygiene`, `claude-env-cleanup`, `project-cleanup`, `project-scaffold`, `sync-parent`, `sp.upgrade-ri`, `spec-kit-monorepo`.
- **Domain-specific extractors / ops (~10):** Trading — `ea-backtest-analyze`, `ea-version-deploy`, `edge-analysis`, `mt4-monitor`; Restaurant/POS — `recipe-costing-extractor`, `recipe-sop-extractor`, `transight-pos-reporting`; misc — `interview`, `internal-comms`, `meeting-notes`, `supermemo2-scheduling`, `git-workflow`, `blender-mcp`, `blender-realistic-face`, `python-cli-tui`, `pytest-testing-patterns`, `windows-cdrive-cleanup`.

**Standouts (evidence of genuine depth, not filler):**
- `gemini-llm-integration` (400 lines): a 10-item "mistakes to avoid" list, a per-model quota-pool table, and a dated production lesson ("All 5 keys were 429'd on gemini-2.0-flash; switching to 2.5-flash worked immediately. Quota is per-model, not per-key"). This is real operational knowledge, not docs regurgitation.
- `browsing-with-playwright` (155 lines + 4 scripts incl. `mcp-client.py`, `start/stop-server.sh`, `verify.py`): full server lifecycle, the `--shared-browser-context` gotcha, navigation/snapshot/screenshot recipes. A genuine agent hand for the web.
- `recipe-costing-extractor` (68 lines + 2 scripts): deterministic multi-tab Excel → Postgres pipeline, idempotent upserts, column-shift gotchas, money-as-integer-cents convention, SQL verify block. Production-grade domain ETL.
- `whatsapp-integration` (421 lines + 7 files): architecture diagram, Go bridge build with version-fix steps, REST API, SQLite read model, FastAPI service, auto-reply — a complete messaging channel.
- `multi-tenant-security` (58 lines): tight enforceable security rule (every query needs `student_id` filter) with correct/violation examples and a review checklist — exactly the kind of guardrail an autonomous FTE needs.

**Genuinely useful agent capabilities** (high reuse): browser automation, the four document skills, gmail/whatsapp comms, the DB/auth/multi-tenant guardrails, gemini/anthropic API patterns. **Trivial or niche** (low cross-product reuse): single-purpose CSS/pagination tips (`tailwind-css-colors`, `react-pagination`), highly personal/domain skills (`mt4-monitor`, `blender-realistic-face`, `supermemo2-scheduling`, `interview`), and OS-cleanup utilities.

## 3. Planned but missing
- README catalog is stale — documents 21 skills; 71 exist. No generated index.
- No versioning, changelog, or dependency manifest per skill (scripts assume ambient `openpyxl`, `psycopg2`, `go`, `npx`, etc. with no declared install contract).
- No skill registry/metadata schema beyond frontmatter `name`/`description`; no tags, maturity flags, or owner.
- `skill-validator` exists but there's no evidence it runs as CI across the catalog.
- Loose `*.md` / `*.skill` duplicates in root suggest an in-progress migration to the directory format that was never finished.

## 4. The AI gap
- The collection is *about* enabling AI, but it is itself a static asset — no agent curates, scores, dedupes, or auto-discovers the right skill for a task.
- No telemetry: nothing records which skills fire, succeed, or fail in production, so quality is asserted (via embedded lessons) rather than measured.
- Skill selection currently relies on the host agent matching the `description` field; there's no semantic router or capability-coverage map.
- Embedded knowledge can silently rot (model names, API quotas, library versions move) with no freshness check or self-update loop.

## 5. Missing pieces to make it sellable / platform-grade
- **Capability store / marketplace:** a registry with searchable metadata, categories, maturity tier (experimental/stable/deprecated), and per-skill owner — turn 71 folders into a browsable catalog.
- **Quality bar:** a `skill-validator` CI gate enforcing frontmatter schema, dead-link/script-smoke checks, and a "lesson must cite a real source/date" rule; auto-fail the stray duplicate-file pattern.
- **Dependency contracts:** each skill declares its runtime deps and required MCP servers/secrets so an FTE can self-provision.
- **Docs regeneration:** auto-build the README skill table from frontmatter so the catalog never goes stale.
- **Trust/safety review** for skills that send email/WhatsApp or touch production DBs (the comms + multi-tenant skills) before they're broadly enabled.

## 6. Native-AI + Autonomous-FTE upgrade
This is the **capability moat** — the skills library is what gives every Aevum autonomous FTE *hands*. An FTE without skills can only talk; with this library it can drive a browser, read/write Office docs and PDFs, send a real email or WhatsApp, run multi-tenant-safe DB queries, deploy to Vercel/Azure, and run domain ETL. Upgrade path:
- **Reuse across every product:** treat the library as a shared dependency injected into all FTEs (Sales FTE → gmail + browsing + docx; Ops FTE → recipe/POS extractors + xlsx; Eng FTE → DB/auth/deploy skills). One library, N personas.
- **Self-improving loop:** add telemetry + an agent that proposes new skills from recurring task patterns and updates stale ones (model/quota/version drift), using `skill-creator-pro` and `skill-validator` as the authoring/QA spine.
- **Semantic capability router:** an embedding index over descriptions so an FTE auto-selects skills instead of relying on keyword match — the difference between "has 71 skills" and "uses the right one autonomously."
- **Guardrails as first-class skills:** `multi-tenant-security` and `context-hygiene` become enforced policy every FTE inherits, not optional reading.

## 7. Showcase angle (for the portal)
- **Headline:** "The hands behind every Aevum FTE — 71 production-tested agent skills, one shared moat."
- 3 stat-benefit bullets:
  - **71 verified skills, ~65 executable scripts** — agents act in the real world (browser, email, WhatsApp, Postgres), not just chat.
  - **Lessons baked in, not looked up** — e.g. the Gemini quota fix that turned 5 dead API keys live; FTEs skip the failures we already paid for.
  - **Write once, reuse everywhere** — the same skill arms a Sales, Ops, or Eng FTE; new capabilities compound across the whole fleet.
- **Demo hook:** Watch one FTE chain skills live — drive a browser to scrape data, drop it into an Excel costing model, and WhatsApp the summary — all from the shared library, no custom code.

## 8. Verdict
- **AI-native score:** 4/5 — purpose-built to extend AI agents and rich with operational knowledge, but the library itself is static (no telemetry, router, or self-update).
- **FTE-fit score:** 5/5 — this is precisely the capability layer autonomous FTEs need; it's the literal difference between an agent that talks and one that does.
- **Maturity:** MVP (top ~15 skills are effectively Production; catalog hygiene and QA gating lag).
- **Recommendation:** **Platform capability** — do not productize standalone. Fund it as the shared moat: add a registry, validator CI, telemetry, and a semantic router, then inject it into every FTE persona across the portfolio.
