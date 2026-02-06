---
name: exe-plan
description: "ONLY when user explicitly types /exe-plan. Never auto-trigger on execute, run, or implement."
argument-hint: "[--sequential]"
---

# /exe-plan - Execute Plan via Subagents

Execute plan.md by dispatching phases to subagents. Main thread orchestrates only.

**Designed for fresh session** — no prior brainstorm or create-plan context needed. plan.md contains everything.

## Flags

- `--sequential`: Force all phases sequential (ignore parallel groups)

## Orchestrator Rules

The main thread is a **lightweight dispatcher**:

- Reads plan.md ONCE at start
- Stores only: phase numbers, dependencies, parallel groups, titles
- Dispatches subagents via Task tool (general-purpose)
- Tracks completion by done-file existence (not contents)
- Reports 1-line progress per phase

The main thread **NEVER**:

- Executes tasks directly
- Reads files beyond plan.md
- Reads done-file contents (only checks existence)
- Accumulates subagent details in its own context

## Execution Flow

```
1. LOAD
   - Read plan.md, verify marker <!-- @plan: /create-plan ... -->
   - Parse phases overview table only (deps, groups, titles)
   - Create plan-results/ if needed

2. SCAN
   - Check plan-results/done-phase-*.md existence
   - Mark completed phases (enables resume after failure)

3. EXECUTE LOOP
   While phases remain:
   a. Find ready phases (deps satisfied, not done)
   b. Batch by parallel group:
      - --sequential: first ready only
      - Default: all ready in same group
      - NEVER mix different groups in parallel
   c. Dispatch subagent(s):
      - One Task call per phase
      - Same group → ONE message with MULTIPLE Task calls (parallel)
   d. On ANY failure: STOP (fail-fast)
   e. Log: "✓ Phase N: [title]"

4. ON SUCCESS
   - Delete plan-results/ directory
   - Delete plan.md
   - Report: "Plan complete. N/N phases succeeded. Cleaned up."

5. ON FAILURE
   - Keep plan-results/ and plan.md intact
   - Report: which failed, what's blocked, "fix and re-run /exe-plan"
```

## Subagent Dispatch

Each subagent receives ONLY:

```markdown
# Context
Goal: [1-2 sentences from plan.md header]
Rationale: [from plan.md Rationale section — approach + pitfalls]
Working directory: [absolute path]
Completed phases: [numbers only]

# Your Assignment: Phase N
[EXACT phase section copied from plan.md]

# Instructions
1. Execute all tasks in order
2. Read codebase files as needed (explore independently)
3. Stay within **Modifies:** scope
4. Write results to plan-results/done-phase-NN.md
5. On failure: stop immediately, return error details
6. Return 1-2 sentence summary

# done-phase-NN.md Format
# Phase N: [title]
**Status:** success|failed
## Tasks
- [x] Task 1 - [brief note]
## Notes
[Anything notable]
```

**Context budget:** Target 50-150 lines per dispatch. Larger means the phase should be decomposed.

## Progress Output

```
/exe-plan

Loading plan.md... N phases, N parallel groups
Resuming: phases 1-3 already complete

Group B (parallel):
  ✓ Phase 2: [title]
  ✓ Phase 3: [title]

Group C (sequential):
  ✓ Phase 4: [title]

Plan complete. N/N phases succeeded. Cleaned up.
```

## Failure Output

```
Plan stopped. N/M phases completed.

Failed: Phase N — [error summary]
Blocked: Phases X,Y,Z

Fix and re-run /exe-plan to resume.
```

## Constraints

- NEVER modify plan.md during execution
- NEVER execute tasks directly — always delegate via Task tool
- Minimal orchestrator context — 1 line per completed phase
- Subagents write ONLY their own done-phase-NN.md
- Same parallel group only — never cross-group parallel
- Fail-fast: stop on first failure, preserve state for resume
- On success: clean delete, not archive
