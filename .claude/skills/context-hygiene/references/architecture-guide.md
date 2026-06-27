# Context Architecture Guide

## Four Tools, Four Loading Patterns

| Tool | When It Loads | Context Cost | Best For |
|------|--------------|-------------|----------|
| **CLAUDE.md** | Session start, every request | Always-on | Stable, always-needed conventions |
| **Skills** | Description at start (~100 tokens); full content on invocation | On-demand | Sometimes-needed domain workflows |
| **Subagents** | When spawned — fresh, isolated context | Zero in main session | Research-heavy, isolated analysis |
| **Hooks** | On activation, run externally | Zero context cost | Deterministic validations |

## The Decision Framework

| Information Type | Best Tool | Rationale |
|---|---|---|
| Always needed, stable | CLAUDE.md | Pay cost once, available everywhere |
| Sometimes needed, stable | Skill | On-demand loading saves context |
| Needs fresh analysis | Subagent | Isolated context prevents pollution |
| Must happen every time | Hook | Deterministic, no LLM variance |

## When to Use Each Tool

### CLAUDE.md — Include when:
- Claude needs it for EVERY task (project conventions, build commands)
- It rarely changes (architectural decisions, team agreements)
- Removing it would cause Claude to make mistakes
- It's under 60 lines total

### Skills — Include when:
- Claude needs it SOMETIMES (domain-specific workflows)
- It's substantial (more than a few lines)
- Manual invocation via `/skill-name` is acceptable
- Content is stable but not universally needed

### Subagents — Employ when:
- Work requires reading many files or extensive research
- Fresh perspective needed without accumulated bias
- Parallel work is desired
- Results can be summarized without losing critical detail

### Hooks — Deploy when:
- Something must happen EVERY time, without exception
- Task is deterministic (no LLM judgment required)
- External execution preferred without context consumption
- Validation, linting, formatting checks

## Target Distribution

| Container | Share | Content Type |
|-----------|-------|-------------|
| CLAUDE.md | ~30% | Always-needed stable conventions |
| Skills | ~25% | Sometimes-needed domain workflows |
| Hooks | ~15% | Deterministic validations |
| Subagents | ~10% | Research-heavy isolated analysis |
| External files | ~20% | Stable reference material |

## Token Budget Allocation

| Component | Recommended % of Context Window |
|-----------|-------------------------------|
| System prompt | 5-10% |
| CLAUDE.md | 5-10% |
| Tool definitions | 10-15% |
| Message history | 30-40% |
| Tool outputs | 20-30% |
| Reserve buffer | 10-15% |

## Common Architecture Mistakes

### Mistake 1: Everything in CLAUDE.md
- **Symptom:** 300+ line CLAUDE.md, important instructions ignored
- **Problem:** Attention diluted across occasionally-relevant content
- **Fix:** Move domain-specific content to Skills; keep CLAUDE.md under 60 lines

### Mistake 2: Never Using Subagents
- **Symptom:** Context fills quickly during research, quality degrades
- **Problem:** All file reads and searches accumulate in main context
- **Fix:** Delegate research to Subagents, receive summaries

### Mistake 3: Skills for Everything
- **Symptom:** Many skills but Claude rarely invokes them correctly
- **Problem:** Skill descriptions don't clearly signal usage opportunities
- **Fix:** Write clear descriptions; use `disable-model-invocation: true` for manual-only

### Mistake 4: Forgetting Hooks Exist
- **Symptom:** Repetitive validation tasks consuming LLM calls
- **Problem:** Using Claude for deterministic checks requiring no reasoning
- **Fix:** Move deterministic validations to Hooks

## Context Savings Math

**Without architecture (everything in CLAUDE.md):**
- 500-line CLAUDE.md: ~4,000 tokens
- Domain frameworks: ~1,500 tokens
- Reference definitions: ~1,000 tokens
- Templates: ~800 tokens
- **Total baseline: ~7,300 tokens/request**

**With architecture:**
- 50-line CLAUDE.md: ~400 tokens (always)
- 3 skill descriptions: ~150 tokens (always)
- Skill content: ~3,300 tokens (only when invoked)
- Research via subagent: 0 tokens in main context
- **Total baseline: ~550 tokens/request**

**Result: ~13x reduction in baseline context load.**

## Dirty Slate vs Clean Context

### The Problem
Sequential agents accumulate irrelevant context:
```
Agent A → (context accumulates) → Agent B → (more accumulates) → Agent C
```
By Agent C, 50,000+ tokens of accumulated process when only ~2,000 tokens of actual input needed.

### The Solution: Clean Context Pattern
```
Orchestrator → [fresh context] → Agent A → [structured summary] → Orchestrator
Orchestrator → [fresh context] → Agent B → [structured summary] → Orchestrator
Orchestrator → synthesizes summaries → final output
```

### Three Subagent Patterns
1. **Stateless** — Fresh isolated context, summary back to orchestrator (best isolation)
2. **Stateful** — Context transfers between agents (dirty slate; use only when genuinely needed)
3. **Shared** — Common memory layer (file/DB) agents read/write (best for long-running projects)
