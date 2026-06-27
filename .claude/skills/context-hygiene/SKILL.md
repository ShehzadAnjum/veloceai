---
name: context-hygiene
description: "Audit and remediate context quality issues in any Claude Code project. Diagnoses context rot (poisoning, distraction, confusion, clash, staleness, accumulation), performs signal-vs-noise audits on CLAUDE.md, assesses architecture distribution across CLAUDE.md/Skills/Subagents/Hooks, checks position optimization, analyzes token budgets, and generates prioritized remediation plans. Use when: (1) CLAUDE.md exceeds 60 lines, (2) Claude starts ignoring instructions or producing inconsistent output, (3) context fills too quickly, (4) starting a new project and want optimal context architecture, (5) session quality degrades over time, (6) user asks to 'audit context', 'clean up CLAUDE.md', 'fix context issues', or 'optimize context'."
---

# Context Hygiene

Audit and remediate context quality issues in Claude Code projects. Run a structured diagnostic across six dimensions, then produce a prioritized remediation plan.

## Audit Workflow

The audit runs six phases sequentially. Present findings after each phase before proceeding.

### Phase 1: Context Rot Diagnosis

Scan CLAUDE.md files (all levels), memory files, and project configuration for all six rot types.

For each file, check:
- **Poisoning** — references to renamed/removed files, outdated patterns, superseded decisions
- **Distraction** — generic advice, verbose explanations, off-topic content
- **Confusion** — ambiguous terminology, similar-sounding concepts without disambiguation
- **Clash** — contradictory instructions (e.g., "always use X" vs "prefer Y" elsewhere)
- **Accumulation** — duplicate or overlapping instructions saying the same thing differently
- **Staleness** — instructions referencing features, files, or patterns that no longer exist

For detailed symptoms and detection patterns, read [references/context-rot-taxonomy.md](references/context-rot-taxonomy.md).

**Output per file:** List each issue found with rot type, line number, the problematic text, and severity (critical/warning/info).

### Phase 2: Signal vs Noise Audit

Apply the 4-question filter to every instruction in CLAUDE.md:

1. Would Claude ask about this without the instruction?
2. Could Claude learn this from existing project files?
3. Does this change frequently (volatile)?
4. Is this a default convention Claude already knows?

Classify each instruction as **signal** or **noise** with reasoning. For the full framework and scoring criteria, read [references/signal-audit-framework.md](references/signal-audit-framework.md).

**Output:** Table of instructions with signal/noise classification, estimated token cost, and recommendation (keep/remove/externalize).

### Phase 3: Position Optimization

Check CLAUDE.md against the U-shaped attention curve:

- **Zone 1 (first 10%)** — Must contain: critical constraints, identity, non-negotiable rules
- **Zone 2 (middle 80%)** — Should contain: reference material, templates, optional content
- **Zone 3 (last 10%)** — Must contain: workflow instructions, startup procedures

Flag critical rules buried in Zone 2 and reference material occupying Zone 1 or 3.

**Output:** Current zone map with misplacement warnings and suggested reordering.

### Phase 4: Architecture Assessment

Evaluate whether information lives in the optimal container. Read [references/architecture-guide.md](references/architecture-guide.md) for the decision framework and distribution targets.

For each block of content in CLAUDE.md, determine if it belongs in:
- **CLAUDE.md** — always needed, stable, under 60 lines total
- **Skill** — sometimes needed, substantial, domain-specific
- **Subagent** — research-heavy, benefits from isolation
- **Hook** — deterministic, must happen every time
- **External file** — stable reference, read on demand

**Output:** Migration table showing each content block, current location, recommended location, estimated token savings, and migration priority.

### Phase 5: Token Budget Analysis

Estimate current baseline context cost:
- Count CLAUDE.md lines and estimate tokens (~8 tokens/line)
- Count skill descriptions loaded at startup
- Identify always-loaded content vs on-demand content
- Compare against target: CLAUDE.md under 60 lines (~480 tokens)
- Calculate potential reduction ratio

**Output:** Current vs target token budget with reduction percentage.

### Phase 6: Remediation Plan

Synthesize all findings into a prioritized action plan:

1. **Critical fixes** — Clashing instructions, poisoned references, stale content pointing to non-existent files
2. **High-impact moves** — Content blocks to migrate from CLAUDE.md to Skills or external files
3. **Position fixes** — Critical rules to move to Zone 1/3
4. **Noise removal** — Instructions to delete entirely
5. **Architecture improvements** — Subagents or Hooks to create

For each action, specify: what to change, where, why, and estimated token savings.

## Report Template

Present the final audit report in this structure:

```
# Context Hygiene Audit Report

## Summary
- Total issues found: N
- Critical: N | Warning: N | Info: N
- CLAUDE.md lines: N (target: <60)
- Estimated noise ratio: N%
- Potential token reduction: N%

## Phase 1: Context Rot
[Issues table]

## Phase 2: Signal vs Noise
[Classification table with recommendations]

## Phase 3: Position Optimization
[Zone map with warnings]

## Phase 4: Architecture Assessment
[Migration table]

## Phase 5: Token Budget
[Current vs target comparison]

## Phase 6: Remediation Plan
[Prioritized action list]
```

## Quick Audit Mode

When the user asks for a quick check rather than a full audit, run only Phases 1 and 2 (rot diagnosis + signal audit) and provide a summary with the top 5 most impactful fixes.

## Post-Remediation Verification

After the user applies fixes, re-run the audit to verify:
- CLAUDE.md is under 60 lines
- No remaining rot detected
- Critical rules are in Zone 1 or Zone 3
- Noise ratio is below 30%
- Architecture distribution is within target ranges
