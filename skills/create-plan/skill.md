---
name: create-plan
description: "ONLY when user explicitly types /create-plan. Never auto-trigger on plan, design, or architect."
argument-hint: "[goal description]"
---

# /create-plan - Interactive Plan Creation

Create structured plan.md for multi-phase task execution.

## Flow

1. **Understand Goal** - Ask user, clarify scope/constraints/success criteria
2. **Research** (if needed) - Explore codebase, identify files/patterns/dependencies
3. **Design Phases** - Break into discrete phases, single responsibility, clear deliverable. Target 3-15 phases (decompose if >15)
4. **Map Dependencies** - Identify phase dependencies, group independent phases for parallel, prioritize thread-safe (no file conflicts)
5. **Write plan.md** - Use exact format below, save to **repository root** (NEVER in subdirectories)
6. **Confirm** - Show summary, get approval

## Location Rules (CRITICAL)

**ALL plan artifacts MUST be at repository root:**
- `plan.md` → repo root
- `plan-results/` → repo root
- `was-plan-YYMMDD-HHMM/` → repo root

**NEVER** create plan files in subdirectories (e.g., `nhand/plan.md` is WRONG).

Even if working on nhand-specific tasks, plan.md goes at nmanager/ root.

## plan.md Format

```markdown
<!-- @plan: /create-plan YYMMDD_HHMM -->
# [Goal Title]

**Goal:** [1-2 sentence description]
**Created:** YYYY-MM-DD

## Phases Overview

| Phase | Name | Depends | Parallel Group | Complexity |
|-------|------|---------|----------------|------------|
| 1 | Create skeleton | - | A | SIMPLE |
| 2 | Copy source X | 1 | B | SIMPLE |
| 3 | Copy source Y | 1 | B | SIMPLE |
| 4 | Update X files | 2 | C | MEDIUM |
| 5 | Update Y files | 3 | C | MEDIUM |
| 6 | Final setup | 4,5 | D | SIMPLE |

## Parallel Groups

- **Group A:** Phase 1 (sequential start)
- **Group B:** Phases 2,3 (safe parallel - different directories)
- **Group C:** Phases 4,5 (safe parallel - no file overlap)
- **Group D:** Phase 6 (sequential end)

## Phase Details

### Phase 1: Create skeleton [SIMPLE]
**Depends:** none
**Parallel:** Group A
**Tasks:**
- Task 1
- Task 2

### Phase 2: Copy source X [SIMPLE]
**Depends:** 1
**Parallel:** Group B
**Modifies:** src/client/ only
**Tasks:**
- Task 1
- Task 2

### Phase 3: Copy source Y [SIMPLE]
**Depends:** 1
**Parallel:** Group B
**Modifies:** src/server/ only
**Tasks:**
- Task 1
- Task 2

[... remaining phases ...]
```

**Note:** `**Modifies:**` helps /exe-plan verify parallel safety.

## Rules

### Phase Design (Subagent-Friendly)
- One clear objective per phase
- Tasks are concrete, actionable
- No task spans multiple files unless tightly coupled
- Verification criteria if phase has testable output
- **Self-contained context**: Each phase includes enough detail for subagent execution WITHOUT reading other phases
- **Explicit file paths**: Name specific files/directories
- **No cross-phase references**: Don't say "use output from Phase 3" - describe what that output is

### Dependency Mapping
- Explicit numeric dependencies (e.g., "Depends: 2,3")
- Transitive deps not repeated (if 3→2→1, phase 3 lists only "2")
- Document what dependency provides: "Depends: 2 (provides lib/core.py)"

### Parallel Safety (CRITICAL)
- Phases in same parallel group MUST NOT touch same files
- When uncertain, make sequential (safety > speed)
- Document WHY parallel is safe: "(safe: operates on src/client/ only)"
- Each phase lists which files/dirs it modifies

### Parallel Group Assignment
- Group by dependency level, not similarity
- Same dependencies + no file conflicts → same group
- Sequential when: output feeds input, same files modified, order matters

### Complexity Labels
- SIMPLE: 1-3 tasks, straightforward
- MEDIUM: 4-8 tasks, some logic
- COMPLEX: 9+ tasks, significant logic (consider decomposing)

### Size Limits
- Max 20 tasks per phase (decompose if more)
- Max 15 phases per plan (create sub-plans if more)

### Context Efficiency
- Keep phase descriptions concise but complete
- Subagent reads codebase files as needed
- Include "Reference:" section for complex phases pointing to relevant files

## Output

Save plan.md, report to user:
```
Plan created: plan.md
- Phases: 6
- Parallel groups: 4 (phases 2-3 and 4-5 can run parallel)
- Estimated complexity: 2 SIMPLE, 2 MEDIUM, 2 SIMPLE

Run /exe-plan to execute.
```

## Plan Lifecycle

```
/create-plan → plan.md created → /exe-plan → plan-results/ with done-phase-*.md
    ↓
On completion: BOTH plan.md AND plan-results/* archived to:
    was-plan-YYMMDD-HHMM/
    ├── plan.md
    ├── done-phase-01.md
    └── ...
```

**Note:** After successful /exe-plan, plan.md is MOVED (not copied) to archive. Working directory clean for next plan.

## Integration with /exe-plan

plan.md structured for optimal subagent execution:

### What Orchestrator Reads
- Marker line (validates plan format)
- Goal header (1-2 sentences for subagent context)
- Phases Overview table (dependency graph, parallel groups)
- Phase headers only (not full task lists) for progress tracking

### What Subagents Receive
Each subagent gets ONLY its assigned phase section:
- Phase title and complexity
- **Depends:** (for context, already verified by orchestrator)
- **Modifies:** (scope constraint - subagent stays within this)
- **Tasks:** (the actual work)
- **Reference:** (optional pointers to relevant files)

### Field Requirements

| Field | Required | Why |
|-------|----------|-----|
| Phase title + complexity | ✓ | Identity, sizing |
| **Depends:** | ✓ | Execution order |
| **Tasks:** | ✓ | Work definition |
| **Parallel:** | Recommended | Enables parallel execution |
| **Modifies:** | Recommended | Prevents file collisions, focuses subagent |
| **Reference:** | Optional | Helps subagent find context quickly |

### Subagent-Friendly Writing

```markdown
# GOOD - Self-contained, explicit
## Phase 4: Create storage module [SIMPLE]
**Depends:** 3 (auth module now exists)
**Modifies:** src/lib/storage.py
**Tasks:**
- [ ] Create read_data(filepath, key) using auth module functions
- [ ] Create write_data(filepath, key, data) with JSON envelope
**Reference:** src/lib/auth.py for available functions

# BAD - Requires reading other phases
## Phase 4: Create storage layer [SIMPLE]
**Depends:** 3
**Tasks:**
- [ ] Use functions from previous phase
- [ ] Follow the same pattern
```
