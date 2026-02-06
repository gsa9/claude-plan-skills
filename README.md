# Claude Plan Skills

For anyone who may find this useful—I wanted to share my planning-first workflow for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). I run Claude Code with `bypassPermissions` mode enabled—this skips all permission prompts and lets Claude work autonomously. **Use with caution:** this setting allows Claude to modify files and execute commands without approval.

This repo intentionally excludes `settings.json` to avoid overwriting your existing configuration. If you want to enable bypass mode, add to your `~/.claude/settings.json`:
```json
{
  "permissions": {
    "defaultMode": "bypassPermissions"
  }
}
```
For a less aggressive option, use `"acceptEdits"` instead—it auto-approves file edits but still prompts for other operations. See the [Hooks](#hooks) section below for a custom statusline you may also want to add.

Three skills, clear roles: **Think → Plan → Do**. `/brainstorm` guides you from open question to clear decision. `/create-plan` turns that decision into a structured plan.md. `/exe-plan` executes it via subagents in a fresh session. The plan is the only artifact that crosses sessions—and it's deleted on success. Code changes are the deliverable, not the plan.

I've also included `/trim`, a skill I've evolved over time that condenses documentation files for token-efficiency.

## Why Custom Plan Skills?

Claude Code has a [built-in Plan Mode](https://docs.anthropic.com/en/docs/claude-code) that I also use. These custom skills are an alternative I reach for when I want my own control over multi-phase plans—exploring options and pitfalls with `/brainstorm` before committing, then creating structured plans with `/create-plan` that break work into self-contained phases for parallel execution.

### The `/clear` Tip

Claude Code's Plan Mode clears context before execution—a smart design. With these skills, you get the same benefit: after `/create-plan` produces your `plan.md`, run `/clear` before `/exe-plan`. This gives the orchestrator a fresh context window with maximum tokens available for subagent dispatch. plan.md carries all the context forward—the decision rationale, the phase details, everything subagents need.

**Recommended flow:**
```
/brainstorm → converge on direction → /create-plan → review plan.md → /clear → /exe-plan
```

### Related Tools and Inspiration

I was already using multi-phase planning skills when I discovered [GSD (Get Shit Done)](https://github.com/glittercowboy/get-shit-done). What caught my attention was GSD's use of subagents to preserve the main context window. I incorporated that pattern into my existing workflow, refining `/exe-plan` so the orchestrator stays minimal while subagents handle the heavy lifting.

Interestingly, Claude Code's built-in Plan Mode uses similar tactics—we're all converging on the same insight: **delegate context consumption to isolated agents**.

GSD does more than I need for my workflow. `/brainstorm`, `/create-plan`, and `/exe-plan` take a lighter approach for developers who prefer tighter feedback loops—brainstorming first, refining plans conversationally, steering direction as needed. More elaborate plans consume more of your usage allowance, which matters on Claude Code's Max plan where token efficiency is important.

## Installation

```bash
git clone https://github.com/gsa9/claude-plan-skills.git ~/.claude
```
Or cherry-pick specific files into your existing config.

## Skills

Custom slash commands invoked with `/<skill-name>`.

### Planning Workflow

| Skill | Purpose |
|-------|---------|
| `/brainstorm` | Think: explore, advise, converge on direction (quick: `/brst`, `/bstorm`) |
| `/create-plan` | Plan: design structured phases with full context for fresh-session execution |
| `/exe-plan` | Do: execute via parallel subagents, delete plan on success |

**Iterative by Design**

Software development has always been iterative—you discover what you need day by day, not upfront. This workflow preserves that rhythm: smaller plans with controlled execution, letting you advance confidently while maintaining the ability to course-correct when a step proves wrong.

**Why This Approach Works**

Since ChatGPT's public release on November 30, 2022, through the transition to Claude Code with Opus on the Max plan, one pattern has proven reliable: tight feedback loops with human judgment at decision points. This workflow gives you end-to-end execution from idea to completion while keeping you in control of pace and direction.

**Flow:** `/brainstorm` → explore options, surface pitfalls, converge on ONE direction → `/create-plan` → structured plan.md with rationale → `/clear` → `/exe-plan` → subagents execute → plan deleted

<details>
<summary><strong>Workflow Overview</strong></summary>

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    PLANNING WORKFLOW: THINK → PLAN → DO                    │
└─────────────────────────────────────────────────────────────────────────────┘

  ┌─────────────┐         ┌─────────────┐         ┌─────────────┐
  │ /brainstorm │         │/create-plan │         │  /exe-plan  │
  │             │         │             │         │             │
  │   THINK     │────────▶│    PLAN     │────────▶│     DO      │
  │             │         │             │         │             │
  └─────────────┘         └─────────────┘         └─────────────┘
        │                       │                       │
        ▼                       ▼                       ▼
  ┌─────────────┐         ┌─────────────┐         ┌─────────────┐
  │ • Explore   │         │ • Rationale │         │ • Subagents │
  │ • Advise    │         │ • Phases    │         │ • Parallel  │
  │ • Converge  │         │ • Deps      │         │ • Resume    │
  │ • Bridge    │         │ • Parallel  │         │ • Cleanup   │
  └─────────────┘         └─────────────┘         └─────────────┘
        │                       │                       │
        ▼                       ▼                       ▼
  ┌─────────────┐         ┌─────────────┐         ┌─────────────┐
  │  Decision   │         │   plan.md   │         │   Deleted   │
  │  (in chat)  │         │   (bridge)  │         │  (cleanup)  │
  └─────────────┘         └─────────────┘         └─────────────┘

  USER CONTROL POINTS
  ───────────────────
  [1] During /brainstorm → Refine direction with Claude, converge when ready
  [2] After plan.md     → Review phases, adjust, /clear before execution
  [3] During /exe-plan  → Monitor progress, fail-fast preserves state for fixes

  WHY THREE STEPS?
  ────────────────
  • /brainstorm thinks WITH you — Claude advises, you decide together
  • /create-plan captures everything in plan.md — the sole context bridge
  • /exe-plan runs fresh — subagents get isolated context, main thread stays lean

  TOKEN LIFECYCLE
  ───────────────
  brainstorm + create-plan share a session (thinking builds on itself)
  /clear wipes context before exe-plan (plan.md carries everything forward)
  On success: plan.md + plan-results/ deleted (code is the deliverable)
```

</details>

---

**Why brainstorm first?** I adopted the idea of prompting Claude to brainstorm from Boris Cherny (then engineer at Anthropic) who describes it in one of his YouTube interviews.

The key evolution: `/brainstorm` isn't just analysis anymore—it's a guided conversation where Claude actively participates. It explores options, gives its honest recommendation, then works with you through interactive dialogue until you both converge on ONE direction. The output is a compact decision block that feeds directly into `/create-plan`'s Rationale section.

In Claude Code, one can use skills in two ways:
- **Start of conversation** - Begin with just `/brainstorm` to open an interactive session, bouncing ideas back and forth until direction crystallizes
- **Anywhere in a prompt** - Include `/brainstorm` at the end or mid-prompt to get structured analysis before committing to implementation

**Tip:** `/brst` and `/bstorm` are short aliases for `/brainstorm`. I often find myself mid-prompt or at the end of a thought when I realize I want analysis before diving into implementation. Having quick aliases means I can append them without breaking my flow or spelling out the full word.

---

<details>
<summary><strong>/brainstorm Flow Diagram</strong></summary>

```
┌─────────────────────────────────────────────────────────────────┐
│             /brainstorm - Think → Advise → Converge             │
└─────────────────────────────────────────────────────────────────┘

  USER INPUT                    CLAUDE ACTIONS                 OUTPUT
  ──────────                    ──────────────                 ──────

  "I want to add              ┌─────────────────┐
   authentication"    ───────▶│  1. EXPLORE     │
                              │  • Ask topic    │
                              │    (if needed)  │
                              │  • Read files   │
                              │  • Surface 2-4  │
                              │    approaches   │
                              │  • Map pitfalls │
                              └────────┬────────┘
                                       │
                              ┌────────▼────────┐
                              │  2. ADVISE      │
                              │  Claude gives   │
                              │  its honest     │
                              │  recommendation │
                              │  • "I recommend │
                              │    X because..."│
                              │  • Cites code   │
                              │  • Flags risks  │
                              └────────┬────────┘
                                       │
                              ┌────────▼────────┐
                              │  3. CONVERGE    │◀──── INTERACTIVE
                              │  Dialogue:      │      (multiple turns)
  "what about Y?" ──────────▶│  • User + Claude│
  "I prefer Z"   ──────────▶│    refine       │
  "good point"   ──────────▶│  • Narrow to ONE│
                              │  • Claude flags │
                              │    concerns     │
                              └────────┬────────┘
                                       │
                              ┌────────▼────────┐
                              │  4. BRIDGE      │──────────────────────┐
                              │  Compact output │                      │
                              │  when aligned   │                      ▼
                              └────────┬────────┘     ┌────────────────────────────┐
                                       │              │ ## Decision: [topic]       │
                              ┌────────▼────────┐     │ **Approach:** JWT + middle │
                              │  Suggest:       │     │ **Why:** fits existing... │
                              │  "/create-plan" │     │ **Pitfalls:** token exp.. │
                              │  when ready     │     │ **Scope:** auth only, no..│
                              └─────────────────┘     └────────────────────────────┘

  CONSTRAINTS
  ───────────
  ✗ No implementation   - thinking only, no file changes
  ✓ Claude participates - recommends, warns, pushes back
  ✓ Lean output         - prose over tables, no token bloat
  ⏸ User controls pace  - never auto-chains to /create-plan
  ◉ Converge or park    - ONE direction, or explicitly pause
```

</details>

---

<details>
<summary><strong>/create-plan Flow Diagram</strong></summary>

```
┌─────────────────────────────────────────────────────────────────┐
│              /create-plan - Design Self-Contained Phases         │
└─────────────────────────────────────────────────────────────────┘

  USER INPUT                    CLAUDE ACTIONS                 OUTPUT
  ──────────                    ──────────────                 ──────

  "Build new auth             ┌─────────────────┐
   system"            ───────▶│ 1. GOAL         │
  (or brainstorm              │  Use decision   │
   decision block)            │  block if avail │
                              └────────┬────────┘
                                       │
                              ┌────────▼────────┐
                              │ 2. RESEARCH     │
                              │  • Read files   │
                              │  • Find patterns│
                              │  • Map deps     │
                              └────────┬────────┘
                                       │
                              ┌────────▼────────┐
                              │ 3. DESIGN       │
                              │    PHASES       │
                              │  • 3-15 phases  │
                              │  • Single resp. │
                              │  • Self-contain │
                              └────────┬────────┘
                                       │
                              ┌────────▼────────┐
                              │ 4. MAP          │
                              │    DEPENDENCIES │
                              │  • Parallel safe│
                              │  • No collisions│
                              └────────┬────────┘
                                       │
                              ┌────────▼────────┐
                              │ 5. WRITE        │──────────────────────┐
                              │    plan.md      │                      │
                              └────────┬────────┘                      ▼
                                       │               ┌───────────────────────────┐
                              ┌────────▼────────┐      │ <!-- @plan: ... -->       │
                              │ 6. CONFIRM      │      │ # Goal Title              │
                              │  Show summary   │      │                           │
                              │  Suggest /clear │      │ ## Rationale              │
                              │  then /exe-plan │      │ **Approach:** JWT + ...   │
                              └─────────────────┘      │ **Why:** fits existing... │
                                                       │ **Pitfalls:** token...    │
                                                       │                           │
                                                       │ | Phase | Depends | Group │
                                                       │ |-------|---------|-------|
                                                       │ | 1     | -       | A     │
                                                       │ | 2     | 1       | B     │
                                                       │                           │
                                                       │ ## Phase 1: ... [SIMPLE]  │
                                                       │ **Depends:** none         │
                                                       │ **Modifies:** src/auth/   │
                                                       │ **Tasks:**                │
                                                       │ - [ ] Create base module  │
                                                       └───────────────────────────┘

  PLAN.MD IS THE CONTEXT BRIDGE
  ─────────────────────────────
  Everything a fresh /exe-plan session needs is IN the plan:
  ✓ Rationale   - why this approach, what pitfalls to avoid
  ✓ Phases      - self-contained, no cross-phase references
  ✓ File paths  - explicit, never "the file from Phase 2"
  ✓ Context     - patterns, conventions subagents need to know

  PHASE DESIGN RULES
  ──────────────────
  ✓ Single responsibility     - one clear objective per phase
  ✓ Self-contained context    - subagent needs no other phases to execute
  ✓ Explicit file paths       - "src/auth/login.ts" not "the login file"
  ✓ Parallel safety           - same group = no file conflicts

  COMPLEXITY LABELS
  ─────────────────
  SIMPLE  │ 1-3 tasks, straightforward
  MEDIUM  │ 4-8 tasks, some logic
  COMPLEX │ 9+ tasks (consider decomposing)

  SIZE LIMITS
  ───────────
  Max 20 tasks/phase · Max 15 phases/plan (sub-plans if more)
```

</details>

---

<details>
<summary><strong>/exe-plan Flow Diagram</strong></summary>

```
┌─────────────────────────────────────────────────────────────────┐
│              /exe-plan - Orchestrated Execution                 │
└─────────────────────────────────────────────────────────────────┘

  DESIGNED FOR FRESH SESSION — no prior context needed.
  plan.md contains everything.

                    ORCHESTRATOR (main context)
                    ───────────────────────────
                              │
                    ┌─────────▼─────────┐
                    │ 1. LOAD plan.md   │
                    │   • Parse phases  │
                    │   • Map deps      │
                    │   • Store minimal │◀─── keeps context LEAN
                    └─────────┬─────────┘
                              │
                    ┌─────────▼─────────┐
                    │ 2. SCAN EXISTING  │
                    │   plan-results/   │◀─── enables RESUME
                    │   done-phase-*.md │
                    └─────────┬─────────┘
                              │
          ┌───────────────────▼───────────────────┐
          │           3. EXECUTE LOOP             │
          │  ┌─────────────────────────────────┐  │
          │  │ Find ready phases (deps met)   │  │
          │  └───────────────┬─────────────────┘  │
          │                  │                    │
          │  ┌───────────────▼─────────────────┐  │
          │  │ Group by parallel safety       │  │
          │  │ Same group → dispatch together │  │
          │  └───────────────┬─────────────────┘  │
          │                  │                    │
          │      ┌───────────┴───────────┐        │
          │      ▼                       ▼        │
          │ ┌─────────┐           ┌─────────┐     │
          │ │Subagent │           │Subagent │     │    PARALLEL
          │ │Phase 2  │           │Phase 3  │     │    EXECUTION
          │ └────┬────┘           └────┬────┘     │
          │      │                     │          │
          │      ▼                     ▼          │
          │ ┌─────────┐           ┌─────────┐     │
          │ │done-02  │           │done-03  │     │
          │ │  .md    │           │  .md    │     │
          │ └─────────┘           └─────────┘     │
          │                  │                    │
          │  ┌───────────────▼─────────────────┐  │
          │  │ Log: "✓ Phase N: [title]"      │  │
          │  │ (1 line only, no details)      │  │
          │  └───────────────┬─────────────────┘  │
          │                  │                    │
          │         [continue until done]         │
          └───────────────────┬───────────────────┘
                              │
                    ┌─────────▼─────────┐
                    │ 4. CLEANUP        │
                    │  • Delete plan-   │
                    │    results/       │
                    │  • Delete plan.md │
                    │  • Report summary │◀─── code is the deliverable
                    └───────────────────┘

  CONTEXT EFFICIENCY
  ──────────────────
  ORCHESTRATOR          │ SUBAGENT
  ──────────────────────│─────────────────────
  • Reads plan ONCE     │ • Gets ONLY its phase
  • Tracks phase #s     │ • Gets Rationale (pitfalls)
  • Never executes      │ • Explores codebase
  • Stays minimal       │ • Writes done file
  • 1 line per phase    │ • Returns summary

  FAIL-FAST + RESUME
  ──────────────────
  ✗ Any failure → STOP immediately
  ✓ Progress preserved in plan-results/
  ✓ Re-run /exe-plan → resumes from failure point

  ON SUCCESS → DELETE
  ───────────────────
  plan.md and plan-results/ are deleted.
  The code changes are what matter.

  FLAGS
  ─────
  --sequential │ Force one-at-a-time (ignore parallel groups)
```

</details>

---

**Features:** Phases with dependencies · Parallel groups · Fail-fast with resume · Rationale section for decision context · Clean delete on success

**Location Rules:** All plan artifacts live at repository root—never in subdirectories:
- `plan.md` → repo root
- `plan-results/` → repo root

**Token-Efficient Lifecycle:** plan.md is disposable. It exists to bridge the gap between the thinking session (brainstorm + create-plan) and the execution session (exe-plan). On success, both plan.md and plan-results/ are deleted. The code changes are the deliverable—re-planning is trivial when needed.

**Context Efficiency:** The orchestrator (main context) stays lean by delegating work to subagents. Each phase runs in isolated context, so complex plans execute without hitting limits. The Rationale section in plan.md gives subagents awareness of the "why" and pitfalls without the orchestrator having to relay brainstorm context.

**Fast execution:** With permissions pre-approved, execution is fast and—when you're confident—can run unsupervised to completion.

**Fully autonomous:** Enable `bypassPermissionsModeAccepted` in settings or run `claude --dangerously-skip-permissions` to skip all permission prompts (use with caution—try at your own risk).

Example `plan.md`:
```markdown
<!-- @plan: /create-plan 260124_1430 -->
# Migrate to New API

**Goal:** Replace legacy REST client with new versioned API across client and server.
**Created:** 2026-01-24

## Rationale

**Approach:** Incremental migration — new types first, then client/server in parallel
**Why:** Existing code already uses typed responses; adding new types is non-breaking
**Pitfalls:** Client and server share response types — must update types before either

## Phases Overview

| Phase | Name | Depends | Parallel Group | Complexity |
|-------|------|---------|----------------|------------|
| 1 | Create types | - | A | SIMPLE |
| 2 | Update client | 1 | B | MEDIUM |
| 3 | Update server | 1 | B | MEDIUM |
| 4 | Integration tests | 2,3 | C | SIMPLE |

## Phase 1: Create types [SIMPLE]
**Depends:** none
**Modifies:** src/types/
**Tasks:**
- [ ] Define new API response types
- [ ] Add validation schemas
```

### /trim

**Why `/trim` deserves a mention:** Documentation and instruction files—especially `CLAUDE.md`—tend to grow over time. Every conversation, Claude reads these files for context. That accumulated verbosity costs tokens without adding value. `/trim` is a careful, conservative tool that condenses prose without losing a single piece of information. It inventories every fact, rule, and example before making changes, then verifies nothing was lost. When in doubt, it keeps the original. This isn't aggressive minification—it's thoughtful tidying that respects your content while reducing what Claude has to parse.

Use `/trim` as a periodic maintenance chore on any documentation that evolves with your project. The result: cleaner files, lower token overhead, same complete information.

<details>
<summary><strong>/trim Flow Diagram</strong></summary>

```
┌─────────────────────────────────────────────────────────────────┐
│               /trim - Token-Efficient Optimization              │
└─────────────────────────────────────────────────────────────────┘

  USER INPUT                    CLAUDE ACTIONS                 OUTPUT
  ──────────                    ──────────────                 ──────

  "/trim CLAUDE.md"    ───────▶┌─────────────────┐
                               │ 1. GET PATH     │
                               │   (arg or ask)  │
                               └────────┬────────┘
                                        │
                               ┌────────▼────────┐
                               │ 2. READ         │
                               │   • Load file   │
                               │   • Note lines  │
                               └────────┬────────┘
                                        │
                               ┌────────▼────────┐
                               │ 3. INVENTORY    │◀─── CRITICAL STEP
                               │   List EVERY:   │
                               │   • Fact        │
                               │   • Rule        │
                               │   • Instruction │
                               │   • Example     │
                               │   • Value       │
                               └────────┬────────┘
                                        │
                               ┌────────▼────────┐
                               │ 4. OPTIMIZE     │
                               │   Condense ONLY:│
                               │   • Filler words│
                               │   • Redundant   │
                               │     phrasing    │
                               │   • Whitespace  │
                               └────────┬────────┘
                                        │
                               ┌────────▼────────┐
                               │ 5. VERIFY       │◀─── MANDATORY
                               │   • Count items │
                               │   • Compare to  │
                               │     inventory   │
                               │   • Check meaning│
                               └────────┬────────┘
                                        │
                        ┌───────────────┴───────────────┐
                        │                               │
                  [counts match]                [counts differ]
                        │                               │
                        ▼                               ▼
               ┌─────────────────┐            ┌─────────────────┐
               │ 6. WRITE        │            │    STOP         │
               │   Save trimmed  │            │   Restore items │
               │   version       │            │   Do not write  │
               └────────┬────────┘            └─────────────────┘
                        │
                        ▼
               ┌─────────────────┐
               │ 7. REPORT       │────────────────────────────────┐
               │   • Before/after│                                │
               │   • What changed│                                ▼
               └─────────────────┘            ┌───────────────────────────────┐
                                              │ Trimmed: CLAUDE.md            │
                                              │ Before: 127 lines             │
                                              │ After:  98 lines              │
                                              │ Condensed:                    │
                                              │  • Verbose explanations       │
                                              │  • Redundant phrases          │
                                              │  • Extra blank lines          │
                                              │ Preserved: all 42 rules/facts │
                                              └───────────────────────────────┘

  SAFETY GUARANTEES
  ─────────────────
  ✓ ZERO information loss   - every fact survives
  ✓ Inventory verification  - counts before and after
  ✓ Meaning preservation    - two rules stay as two rules
  ✓ Fail-safe default       - when uncertain, keep it

  WHAT GETS CONDENSED              WHAT NEVER CHANGES
  ──────────────────────           ───────────────────
  • "In order to" → "to"           • Rules and constraints
  • "It should be noted" → cut     • Examples and values
  • Redundant explanations         • Technical instructions
  • Excessive whitespace           • Code blocks
  • Filler phrases                 • Any distinct meaning

  PHILOSOPHY
  ──────────
  Tokens are cheap. Lost information cannot be recovered.
  When in doubt: KEEP IT.
```

</details>

---

## Hooks

### Status Line

`hooks/statusline.js` displays context window usage (bar + %), current model, working directory:
```
██░░░░░░░░ 20%      Opus 4.5     project
```
No color changes as context grows—intentionally subtle to stay out of your way.

Enable in `settings.json`:
```json
{
  "statusLine": {
    "type": "command",
    "command": "node -e \"require(require('os').homedir() + '/.claude/hooks/statusline.js')\""
  }
}
```

## Co-Author

Built in collaboration with [Claude](https://claude.ai) by Anthropic.

## License

MIT
