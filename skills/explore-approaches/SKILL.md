---
description: Explore multiple implementation approaches in parallel via concurrent agents, then compare
argument-hint: description of the feature or problem to solve
---

# /explore-approaches — Parallel Exploration

Explores 2 concurrent implementation approaches via sub-agents, compares the results, and recommends the best one.

**Goal**: $ARGUMENTS

## Principles

- **Concrete**: each agent produces working CODE, not a plan
- **Timeboxed**: max 15 turns per agent — no exploration spiral
- **Comparable**: same evaluation criteria for each approach
- **User decides**: recommend, don't impose

---

## Phase 1: Scoping (2 minutes max)

1. Read relevant files to understand the existing architecture
2. Identify exactly 2 distinct and viable approaches
3. Present the 2 approaches in 2-3 sentences each to the user:

```
Approach A: [short name] — [description]
Approach B: [short name] — [description]
```

Use `AskUserQuestion` to confirm or adjust the approaches before launching agents.

---

## Phase 2: Parallel Agents

Launch 2 Task agents in parallel with `isolation: "worktree"`:

Each agent receives a structured prompt:
```
Context: [project description and relevant files]
Goal: [the requested feature]
Approach: [A or B — detailed description]
Constraints:
- Implement the approach by modifying the necessary files
- Write at least one test that verifies the behavior
- Run the modified tests to verify they pass
- Summarize: modified files, lines of code added, perceived complexity
```

**Important**: use `max_turns: 15` to timebox each agent.

---

## Phase 3: Comparison

When both agents complete, compare on these criteria:

| Criterion | Approach A | Approach B |
|-----------|-----------|-----------|
| Files modified | N | N |
| Lines added | N | N |
| Tests written | N | N |
| Tests pass? | yes/no | yes/no |
| Consistency with existing code | score /5 | score /5 |
| Complexity | low/medium/high | low/medium/high |

---

## Phase 4: Recommendation

Present the comparison table and recommend the approach that:
1. Has the fewest modified files
2. Is the most consistent with existing project patterns
3. Has passing tests

Use `AskUserQuestion`:
```
label: "Approach A — [name]"
description: "[summary of pros/cons]"
```

---

## Phase 5: Application

After the user's choice:
1. Apply the changes from the chosen worktree into the current branch
2. Re-run tests to confirm
3. Summarize the modified files

Do NOT commit automatically — wait for the user to ask.

---

## Notes

- If both approaches fail (broken tests), say so honestly and suggest a 3rd path
- Don't favor an approach for aesthetic reasons — prioritize what works
- Worktrees are automatically cleaned up if unused
