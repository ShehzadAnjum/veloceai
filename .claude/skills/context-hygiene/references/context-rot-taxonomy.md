# Context Rot Taxonomy

## Four Types of Context Rot

### 1. Poisoning — Outdated Information Persists

**Cause:** Mid-session changes (renames, design pivots, decision reversals) conflict with earlier discussion still in context.

**Symptoms:**
- Uses outdated terminology or patterns that were previously correct
- References renamed files, functions, or variables by old names
- Applies superseded architectural decisions

**Detection patterns:**
- Search for renamed identifiers still referenced by old name in CLAUDE.md or memory files
- Check for contradicting instructions at different points in the file
- Look for TODO/FIXME comments referencing completed work
- Find references to deleted files or removed APIs

**Fix:**
- Remove or update outdated references immediately
- Use `/clear` if context is heavily poisoned
- Externalize volatile information to files Claude reads on demand rather than CLAUDE.md

### 2. Distraction — Irrelevant Content Dilutes Attention

**Cause:** Tangential discussions, verbose explanations, or off-topic content consuming attention budget.

**Symptoms:**
- Less focused responses
- Missed details in current task
- Tangential considerations appearing in outputs
- Decreased adherence to project-specific rules

**Detection patterns:**
- Instructions that restate what Claude already knows (standard conventions)
- Verbose explanations where a one-liner suffices
- Documentation of removed features or abandoned approaches
- Generic advice not specific to the project

**Fix:**
- Apply 4-question signal audit to every instruction
- Move detailed reference material to external files
- Keep CLAUDE.md under 60 lines

### 3. Confusion — Similar Concepts Conflate

**Cause:** Working with similarly-named entities, overlapping terminology, or ambiguous references.

**Symptoms:**
- Conflating distinct services, files, or concepts
- Wrong terminology application
- Cross-domain pattern misapplication
- Mixing up similar variable/function names

**Detection patterns:**
- Multiple entities with similar names (e.g., `UserService` vs `UserManager`)
- Overloaded terms used in different contexts without disambiguation
- Abbreviations that could refer to multiple things

**Fix:**
- Add explicit disambiguation notes for similar-sounding concepts
- Use full qualified names in instructions
- Group related concepts together with clear boundaries

### 4. Clash — Contradictory Instructions Compete

**Cause:** Instructions added at different times that now contradict each other.

**Symptoms:**
- Inconsistent decisions across outputs
- Requests for clarification on already-decided matters
- Oscillating between two approaches

**Detection patterns:**
- "Always use X" in one place, "prefer Y" in another
- Style rules that conflict (e.g., "use semicolons" vs "no semicolons")
- Workflow instructions that specify different orderings
- Multiple error-handling strategies prescribed

**Fix:**
- Audit for contradictions by reading all instructions sequentially
- Resolve conflicts by keeping the most recent/correct version only
- Add explicit precedence rules if genuine exceptions exist

## Additional Rot Types (from exercises)

### 5. Accumulation — Duplicate/Overlapping Content

**Cause:** Content added over time that duplicates or overlaps with existing instructions.

**Detection:** Multiple instructions saying the same thing in different words.

**Fix:** Deduplicate, keeping the clearest version.

### 6. Staleness — Content No Longer Reflects Reality

**Cause:** Content was accurate when written but the project has evolved.

**Detection:** Instructions referencing features, files, or patterns that no longer exist in the codebase.

**Fix:** Verify each instruction against current codebase state; remove or update stale entries.
