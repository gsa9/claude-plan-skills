---
name: brst
description: "Trigger: /brst, /bstorm, or /brainstorm. Never auto-trigger on think, analyze, or consider."
---

# /brst - Explore → Advise → Converge → Ready

Guided thinking from "what should we do?" to "let's plan it."

## When to Use

- Before building anything non-trivial
- Multiple approaches or unclear requirements
- Surface pitfalls before committing to a path

## Flow

### 1. EXPLORE
- Ask "What needs solving?" (skip if obvious from conversation)
- Read relevant codebase files, understand constraints/patterns
- Surface 2-4 viable approaches with key tradeoffs
- Identify pitfalls and risks for each

### 2. ADVISE
Claude gives its honest recommendation:
- **Be direct** — "I recommend X because..." not "here are some options"
- Cite specific codebase evidence (files, patterns, constraints)
- Flag risks user may not have considered
- Suggest scope adjustments if warranted
- Push back if user's direction has issues — say so early

### 3. CONVERGE
Interactive dialogue — as many turns as needed:
- User and Claude discuss, refine, challenge each other
- Narrow from options to ONE clear direction
- Claude flags concerns, doesn't rubber-stamp
- Either converge on a direction or explicitly park the decision

### 4. BRIDGE
When both sides agree, output:
```
## Decision: [topic]
**Approach:** [1-2 sentences — what we're building and how]
**Why:** [reasons grounded in codebase evidence]
**Pitfalls:** [key risks to guard against]
**Scope:** [what's in, what's explicitly out]
```
Then suggest: "Ready for `/create-plan`?"

Decision block transfers into plan.md's Rationale section — keep it tight.

## Constraints

- **No implementation** — thinking only, no file changes
- **Claude participates** — recommend, warn, push back, don't just list options
- **Lean output** — prose over tables, no token-heavy formatting
- **User controls pace** — never auto-chain to /create-plan
- **Converge or park** — end with ONE direction or explicitly pause
