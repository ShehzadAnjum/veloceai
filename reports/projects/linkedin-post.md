# LinkedIn Post Writer — a Claude Code skill (prompt pack) that drafts on-brand LinkedIn posts on demand

## 1. Snapshot
- **One-liner:** A Claude Code *skill* (markdown instructions + style references) that turns a topic into a formatted, voice-consistent LinkedIn post when invoked inside Claude.
- **Category:** Content / Marketing AI (personal branding / thought-leadership copywriting).
- **Target buyer:** Solo founders, devrel/AI practitioners, consultants, and personal-brand builders who already use Claude Code and want repeatable, high-engagement LinkedIn drafts.
- **Tech stack:** No application code. It is plain Markdown: `SKILL.md` + three `references/*.md` style guides, packaged as a `.skill` zip. A sibling `project-scaffold` skill ships two Python utility scripts (`init_project.py`, `audit_project.py`) for bootstrapping/auditing Claude projects — unrelated to posting. Repo also carries a Playwright MCP config (`.mcp.json`) and a stray browser console log; no posting automation uses it.
- **Maturity:** **Prototype** (as a sellable *product*). As a *skill* it is complete and usable, but it is a static instruction set with zero automation, scheduling, posting, accounts, persistence, or integration. Justification: the entire "product" is ~530 lines of guidance text; the only executable code in the repo belongs to the scaffolding helper, not to LinkedIn.
- **Live URL / demo:** None. Runs only inside a Claude/Claude Code session that has the skill loaded.
- **Path:** /home/anjum/dev/linkedin_post

## 2. What's built today (verified)
Verified by reading the files:
- **`linkedin-post/SKILL.md`** — a well-structured authoring workflow: (1) clarify topic/goal/audience/tone/length, (2) select a post format, (3) write using a fixed section recipe (hook → personal context → reframe → metaphor → arrow-list "meat" → philosophical closer → single CTA → 4-5 hashtags), (4) a 12-point review checklist, (5) offer hook/length variations and posting-time tips. Frontmatter `description` gives strong trigger keywords ("LinkedIn post", "LinkedIn content", etc.).
- **`references/voice-and-style.md`** (174 lines) — the primary style bible: voice rules, "Not X. Y." contrast pattern, a proven default structure ("480+ reactions"), arrow-list formatting, hook/reframe/closer pattern libraries, an explicit anti-pattern list (no bold-unicode, no "I'm excited to announce", no emoji bullets, no in-body links), tone calibration table, and a length guide.
- **`references/hooks-and-examples.md`** (123 lines) — hook formula library (pattern-interrupt, curiosity-gap, vulnerability, authority, contrarian), formatting/"see more" threshold rules, CTA patterns, hashtag strategy, and a 2024-2026 "algorithm signals" + optimal-posting-times cheat sheet.
- **`references/post-formats.md`** (111 lines) — templated formats (Story, Listicle, Hot Take, Achievement, Before/After, Question/Poll) with worked examples.
- **Duplicate install copy** under `.claude/skills/linkedin-post/` (same four files) plus the packaged `linkedin-post.skill` zip — i.e., it is set up to be distributed/loaded as a Claude skill.
- **`project-scaffold`** (separate skill) — bootstraps/audits Claude Code projects; ships real Python (`init_project.py`, `audit_project.py`). It is a generic dev-tooling skill that happens to live in the same folder; **not** part of the LinkedIn content pipeline.

What this means: today the deliverable is high-quality, opinionated *copywriting guidance* that a human invokes one post at a time. The "AI" is whatever Claude session executes the markdown.

## 3. Planned but missing
There is no roadmap file in the repo; the following are obvious, unbuilt capabilities (treat as gaps, not commitments):
- No input ingestion (no way to feed a blog post, transcript, repo diff, or release note and auto-derive a post).
- No idea generation / content calendar / topic pipeline.
- No scheduling or queue.
- No publishing — no LinkedIn API/OAuth, no draft export, no posting via the bundled Playwright MCP (the console log shows it was only used to browse an unrelated docs site).
- No analytics loop (it *describes* algorithm signals but never measures engagement or learns from it).
- No multi-post / campaign management, no brand-profile persistence beyond the static style guide.
- No approval workflow, accounts, or multi-user support.

## 4. The AI gap
- **No autonomous AI of its own.** The skill contains no model calls, no agent loop, no API keys, no orchestration. It is inert text that only does something when a human prompts a Claude session and supplies the topic, reviews the draft, and copies it into LinkedIn manually.
- **Human-in-every-step.** Clarify, choose format, review checklist, post — all manual. It augments a writer for one post; it does not run a content function.
- **No feedback / learning.** The "algorithm signals" and "480+ reactions" claims are baked-in heuristics, not measured. Nothing closes the loop from real engagement back into future drafts.
- **Net:** strong *prompt engineering*, zero *product AI*. It is a prompt pack, not an agent.

## 5. Missing pieces to make it sellable
1. **Wrap it in a runtime** — a hosted agent or app (Claude API + the skill as system prompt) so a buyer doesn't need Claude Code installed.
2. **Inputs** — ingest a URL/transcript/changelog/voice-note and auto-extract post angles.
3. **Publishing** — LinkedIn OAuth + scheduled posting (or a reliable draft-export/Buffer-style integration); the Playwright dependency is fragile and ToS-risky for posting.
4. **Brand memory** — persistent voice profile per user (sample posts → fine-tuned style vector) instead of one generic style file.
5. **Analytics feedback loop** — pull post performance, attribute it, and bias future drafts toward what works.
6. **Approval + guardrails** — review queue, banned-claims/compliance filter, "no fabricated metrics" check.
7. **Calendar/cadence** — generate and schedule N posts/week automatically.
8. **Multi-tenant accounts, billing, UI.**

## 6. Native-AI + Autonomous-FTE upgrade
Reframe from "post writer skill" to an **Autonomous Personal-Brand / Content-Marketer FTE** that runs 24/7:
- **Ideates** — monitors the user's GitHub commits, shipped features, saved articles, and calendar to surface post-worthy moments without being asked.
- **Drafts** — applies this skill's voice/format/anti-pattern rules to produce on-brand posts, plus alt hooks.
- **Schedules & posts** — maintains a 3-5x/week cadence at optimal times, posts via LinkedIn integration.
- **Learns** — ingests engagement data and continuously tunes hooks/topics/timing per the algorithm-signals model.
- **Guardrails/approval** — every post passes a compliance + no-fabricated-claims check and (optionally) a one-tap human approval queue; full audit trail.
- **Pitch:** "Replaces ~8-12 hours/week of a content marketer / ghostwriter" — always-on ideation, drafting, scheduling, and performance optimization for one brand, at a fraction of an FTE.

## 7. Showcase angle (for the portal)
- **Headline:** "Your always-on LinkedIn ghostwriter — on-brand posts, on schedule, while you sleep."
- 3 stat-benefit bullets:
  - **Voice-locked:** a proven structure (480+ reactions on the reference post) with explicit anti-spam guardrails — no bold-unicode, no "excited to announce", no engagement bait.
  - **Cadence on autopilot:** built-in algorithm + optimal-time playbook drives a consistent 3-5 posts/week.
  - **Replaces ~10 hrs/week** of a ghostwriter: ideate → draft → schedule → learn, with human approval optional.
- **Demo hook:** Paste a raw idea (or drop a GitHub repo link) → watch it become three publish-ready LinkedIn variants with hooks, hashtags, and a recommended post time in under 30 seconds.

## 8. Verdict
- **AI-native score:** 1/5 — no autonomous AI in the artifact; it is static guidance executed by an external Claude session.
- **FTE-fit score:** 2/5 — the *domain* (content marketing / personal branding) is a strong autonomous-FTE fit, but nothing here runs unattended; it augments one post at a time.
- **Maturity:** Prototype (excellent prompt pack, no product).
- **Recommendation:** **Incubate.** The copywriting IP is genuinely good and differentiated; it is the right *seed* for an Autonomous Content-Marketer FTE. Wrap it in a runtime with inputs, publishing, memory, and an engagement feedback loop before it can be a showcase product. Not a flagship today; promote to Secondary/Flagship only after the agent layer ships.
