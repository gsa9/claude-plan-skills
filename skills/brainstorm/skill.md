---
name: brainstorm
description: "Trigger: /brst, /bstorm, or /brainstorm. Never auto-trigger on think, analyze, or consider."
---

# /brainstorm - Interactive Analysis Without Implementation

Produce comprehensive analysis without making changes.

## Flow

1. **Ask topic** - "What would you like me to brainstorm about?"
2. **Gather context** - scope, constraints, success criteria (or skip for defaults)
3. **Research** - read relevant files, understand current state
4. **Output analysis** - structured brainstorm with options, pitfalls, recommendations
5. **Wait** - user decides next step (elaborate, implement, plan, or save)

## Output Structure

```markdown
## Brainstorm: [TOPIC]

### Context
[Current state summary]

### Key Findings
| # | Finding | Impact |
|---|---------|--------|

### Options
**Option A/B:** Pros, cons, effort

### Pitfalls
| Pitfall | Cause | Prevention |
|---------|-------|------------|

### Recommendations
1. Primary choice + rationale
2. Alternative approach

### Next Steps
- [ ] Actions if user proceeds
```

## Constraints

- **No implementation** - analysis only, no file changes
- **Cover pitfalls** - most valuable part
- **Multiple options** - never just one approach
- **User decides** - end with options, wait for direction
