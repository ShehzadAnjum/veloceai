# Context Lifecycle Thresholds

## Context Utilization Zones

| Zone | Utilization | Action |
|------|-------------|--------|
| Green | 0-50% | Work freely, plenty of room |
| Yellow | 50-70% | Monitor; prepare compaction strategy |
| Orange | 70-85% | **Compact NOW** — quality degradation begins |
| Red | 85-95% | Emergency compaction required |
| Black | 95%+ | Reset required (`/clear`) |

**Critical thresholds:**
- **50%** = Start monitoring (watch point)
- **70%** = Quality cliff — degradation begins here
- **85%** = Emergency territory
- **95%+** = Graceful compaction impossible; must reset

## /clear vs /compact Decision

**Use `/clear` when:**
- Task is complete and you're starting new work
- Context is poisoned (outdated decisions, confusion from renames)
- Switching to unrelated work
- In Black zone (95%+)
- Session is 3+ days old

**Use `/compact` when:**
- Same task continues and decisions must be preserved
- Context is large but still relevant
- Compaction is faster than re-explaining from scratch

## Three Compaction Strategies (ranked by quality)

### 1. Naive (worst — loses 30-50% quality)
```
/compact Keep the important stuff
```

### 2. Structured (good)
```
/compact Preserve: [list specific decisions, file changes, constraints]
Discard: [exploration, dead-ends, tangential discussions]
Focus: [current task description]
```

### 3. Progress-file (best — preserves 30-50% more than naive)
1. Update progress file with current state, decisions, and remaining work
2. Then run `/compact`
3. Progress file survives compaction as external state

## Seven Token Budgeting Strategies

1. **Summarize large text blocks** — Documents > 2,000 tokens: extract and condense before inclusion
2. **Chunk documents into vector DB** — Query-based retrieval of relevant chunks only
3. **Offload to external memory** — Pass references not full content; use files for state
4. **Use relevancy checks** — Conditional inclusion based on task requirements
5. **Structure prompts wisely** — System messages for stable rules; user messages for task-specific
6. **Monitor real-time** — Check context every 10 messages above 50%; 70%+ = compact; 85%+ = mandatory
7. **Multi-round processing** — Sequential rounds with fresh context per round

## The Save Checkpoint Pattern

1. Work in Green/Yellow zone
2. Make meaningful progress
3. Save checkpoint externally (commit, save document, export notes)
4. Check utilization
5. Compact if needed
6. Continue from checkpoint

## Session Persistence

- `claude --continue` — Resume most recent session
- `claude --resume` — Pick from recent sessions
- `/resume` — Switch conversations in-session

**The 3-Day Rule:** Sessions become unreliable after 3-4 days due to accumulated tangents and implicit assumptions that no longer hold.

## Progress File Template (for long-horizon work)

Seven essential sections:
1. **Completed** — Finished items with session markers
2. **In Progress** — Active tasks with remaining work annotated
3. **Blocked** — Specific blockers with actionable details
4. **Decisions Made** — What, why, alternatives rejected
5. **Known Issues** — Limitations with impact and resolution plans
6. **Session Log** — Narrative per session: challenges, stopping points
7. **Next Steps** — Priority queue for upcoming work

## Session Protocols

### Start protocol (every session):
1. Read progress file for current state
2. Verify deliverable status
3. Select highest-priority incomplete item
4. Establish baseline understanding

### Exit protocol (every session):
1. Save work at stable checkpoint
2. Update progress file (completed tasks, new decisions, discovered issues, session summary)
3. Never end with work in disarray — finish cleanly or annotate remaining work precisely

## The Plan-Clear-Execute Pattern (with Tasks system)

1. **Plan phase:** Create task dependencies (DAG) while context is fresh
2. **Clear phase:** Run `/clear` when context reaches 60-80% usage
3. **Execute phase:** Continue work — task roadmap persists on disk at `~/.claude/tasks/`

Tasks survive `/clear`, crashes, and work cross-session via `CLAUDE_CODE_TASK_LIST_ID`.

## Workflow Drift Prevention

By turn 20, earlier context receives diminished processing attention. Two injection strategies:

| Hook | Fires | Best For |
|------|-------|---------|
| **UserPromptSubmit** | Once per user message | Initial context, one-shot queries |
| **PreToolUse** | Before each tool execution | Ongoing relevance throughout multi-step work |

Use both: UserPromptSubmit for baseline session context, PreToolUse for execution-time relevance.

## Four Production-Readiness Criteria

Score each 1-5. Below 3 on any = not production-ready.

1. **Consistency** — Same quality at turn 1 vs turn 50
2. **Persistence** — Resume after 24h break in < 5 minutes
3. **Scalability** — Handle 10+ step tasks without drift
4. **Knowledge** — Apply domain expertise automatically without reminders
