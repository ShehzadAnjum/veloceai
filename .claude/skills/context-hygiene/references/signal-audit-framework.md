# Signal vs Noise Audit Framework

## The Core Problem

Research finding: **30-60% of tokens sent to models add no value.** These tokens compete with signal for the model's limited attention budget (~150-200 distinct instructions before compliance drops, with ~50 consumed by the system prompt).

## The 4-Question Signal Audit

Apply to EVERY instruction in CLAUDE.md and memory files:

### Question 1: Would Claude ask about this without it?

- **Yes** = Signal. Claude genuinely needs this guidance.
- **No** = Potential noise. Claude would proceed correctly without it.

Example noise: "Write clean, readable code" — Claude does this by default.
Example signal: "Use `pnpm` not `npm` for all package operations" — Claude would default to npm.

### Question 2: Could Claude learn this from existing materials?

- If the information already exists in package.json, tsconfig, README, or other project files = **Noise in CLAUDE.md**
- Claude can read these files on demand; duplicating their content wastes tokens.

Example noise: "This project uses TypeScript 5.3" — visible in package.json.
Example signal: "Run `pnpm test:integration` before submitting PRs" — not discoverable from files alone.

### Question 3: Does this change frequently?

- **Volatile information becomes stale** and creates poisoning.
- Frequently-changing info belongs in external files, not CLAUDE.md.

Example noise: "Current sprint focuses on auth refactor" — stale in 2 weeks.
Example signal: "All API endpoints must return JSON:API format" — stable convention.

### Question 4: Is this a default convention Claude already knows?

- Standard professional practices don't need restating.
- Only include conventions that DIFFER from defaults.

Example noise: "Use meaningful variable names" — standard practice.
Example signal: "Prefix all database column names with table abbreviation: `usr_name`, `ord_total`" — project-specific.

## Signal Categories (KEEP)

- Non-obvious commands and build steps
- Style rules that differ from language/framework defaults
- Approval requirements and workflow gates
- Project-specific naming conventions
- Architectural decisions and their rationale
- Known gotchas and footguns
- Integration-specific configuration

## Noise Categories (REMOVE or EXTERNALIZE)

- Information inferrable from existing project files
- Standard professional conventions any developer follows
- Frequently-changing information (move to external files)
- Detailed reference documentation (move to references/)
- Verbose explanations of obvious concepts
- Generic best practices

## The Three-Zone Position Strategy

LLM attention follows a **U-shaped curve**: high attention at beginning and end, ~30% less recall in the middle.

| Zone | Position | Content Type | Examples |
|------|----------|-------------|----------|
| Zone 1 | First 10% | Critical constraints, identity, non-negotiable rules | "Never commit .env files", "Use pnpm only" |
| Zone 2 | Middle 80% | Reference material, templates, optional content | File structure docs, API patterns |
| Zone 3 | Last 10% | Workflow instructions, startup procedures | "On session start, read tasks.md" |

## Progressive Disclosure

Replace verbose inline content with concise references to external files:

**Before (inline, 25 lines):**
```
## API Error Handling
All API errors must follow this format:
{ "error": { "code": "...", "message": "...", ... } }
[20 more lines of detail]
```

**After (reference, 1 line):**
```
## API Error Handling
Follow the error format in docs/api-errors.md
```

**Target: CLAUDE.md under 60 lines** while maintaining comprehensive guidance through references.

## Audit Scoring

For each instruction, score:
- **Signal strength** (1-5): How much does removing this hurt output quality?
- **Token cost** (low/medium/high): How many tokens does this consume?
- **Stability** (1-5): How long before this becomes stale?

Priority to remove: Low signal + High cost + Low stability.
Priority to keep: High signal + Low cost + High stability.
