---
name: create-plan
description: "ONLY when user explicitly types /create-plan. Never auto-trigger on plan, design, or architect."
argument-hint: "[goal description]"
---

# /create-plan - Structured Plan for Subagent Execution

Create plan.md that a fresh `/exe-plan` session can execute with zero prior context.

plan.md is the **only context bridge** between this session and execution. Everything a subagent needs must be in the plan.

## Flow

1. **Goal** — Clarify what's being built. Use brainstorm decision block if available
2. **Research** — Read codebase as needed, identify files, patterns, dependencies
3. **Design phases** — 3-15 discrete phases, single responsibility each
4. **Map dependencies** — Identify parallel groups, verify no file collisions
5. **Write plan.md** — At **repository root** (NEVER in subdirectories)
6. **Confirm** — Show summary, suggest running `/exe-plan` in a fresh session

## plan.md Format

```markdown
<!-- @plan: /create-plan YYMMDD_HHMM -->
# [Goal Title]

**Goal:** [1-2 sentence description]
**Created:** YYYY-MM-DD

## Rationale

**Approach:** [chosen approach — from brainstorm or stated by user]
**Why:** [key reasons grounded in codebase evidence]
**Pitfalls:** [risks to guard against during execution]

## Phases Overview

| Phase | Name | Depends | Parallel Group | Complexity |
|-------|------|---------|----------------|------------|
| 1 | ... | - | A | SIMPLE |
| 2 | ... | 1 | B | SIMPLE |
| 3 | ... | 1 | B | MEDIUM |
| 4 | ... | 2,3 | C | SIMPLE |

## Phase Details

### Phase 1: [title] [COMPLEXITY]
**Depends:** none
**Parallel:** Group A
**Modifies:** [file/directory scope]
**Tasks:**
- [ ] Task 1
- [ ] Task 2
**Reference:** [relevant files for subagent context]
```

## Phase Design Rules

Each phase executes as an **isolated subagent in a fresh context**. It must be self-contained:

- **Explicit file paths** — name specific files, never "the file from Phase 2"
- **No cross-phase references** — describe what's needed, don't point to other phases
- **Include codebase context** — patterns, naming conventions, constraints the subagent needs to know
- **One clear objective** — single responsibility
- **Concrete tasks** — actionable, unambiguous

### Complexity & Limits

- SIMPLE: 1-3 tasks | MEDIUM: 4-8 tasks | COMPLEX: 9+ (decompose)
- Max 20 tasks/phase, max 15 phases/plan

### Dependencies

- Explicit numeric: "Depends: 2,3"
- No transitive repetition (if 3→2→1, phase 3 lists only "2")
- Document what it provides: "Depends: 2 (creates lib/core.py)"

### Parallel Safety

- Same group MUST NOT touch same files
- When uncertain, make sequential (safety > speed)
- Each phase lists what it modifies via **Modifies:** field

## Location Rules

**ALL plan artifacts at repository root:**
- `plan.md` → repo root
- `plan-results/` → repo root (created by /exe-plan at execution time)

## Output

```
Plan created: plan.md
- Phases: N
- Parallel groups: N (phases X,Y can run parallel)
- Complexity: breakdown

Run /exe-plan in a fresh session to execute.
```

## Lifecycle

```
/brainstorm → /create-plan → plan.md written
    ↓
Fresh session → /exe-plan → executes → deletes plan.md + plan-results/
```

plan.md is disposable. After successful execution, it's deleted — not archived. The code changes are the deliverable, not the plan.
