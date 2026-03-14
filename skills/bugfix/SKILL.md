---
description: Test-first bug diagnosis and fix — reproduce, fix, verify, iterate
argument-hint: bug description (observed symptom)
---

# /bugfix — Test-First Fix

Diagnoses and fixes a bug following a strict cycle: reproduce with a test, fix, verify.

**Bug**: $ARGUMENTS

**Prerequisite**: check the project's CLAUDE.md for test commands and framework conventions.

## Principles

- **Test-first**: NEVER fix without a failing reproduction test first
- **Root cause**: find the real cause, not a workaround
- **Iterative**: if the fix doesn't work, change strategy (no minor variations)
- **Fully autonomous**: iterate until green WITHOUT asking the user. Only stop if all strategies are exhausted

---

## Phase 1: Diagnosis and Test (parallel agents)

Launch 2 Task agents in parallel:

### Agent A — Diagnosis
1. Read relevant files mentioned in the bug description
2. Trace the complete data flow (save → load → render)
3. Check BOTH sides (client AND server)
4. Return: probable cause, suspicious file:line, recommended fix strategy

### Agent B — Reproduction Test
1. Identify the right test file (system test if UI, unit/integration otherwise)
2. Write a minimal test that **reproduces exactly the symptom**
3. Run the test to confirm it fails (use the project's test command)

4. Verify the test fails **for the right reason** (not a setup error)
5. Return: the test and its error output

If Agent B cannot reproduce the bug:
- Use Agent A's findings to adjust the angle
- Try a 2nd different test
- If impossible to reproduce after 2 attempts, report to the user

**Output**: Diagnosis + failing test.

---

## Phase 2: Autonomous Fix Loop

**Do NOT ask the user — iterate until green.**

1. Implement the **minimal** fix — only touch the necessary code
2. Re-run the reproduction test

### If the test still fails:

- **Do NOT** retry the same approach with minor variations
- Analyze WHY the fix didn't work
- Choose a **fundamentally different** strategy
- Retry — loop up to 5 different strategies

### If the test passes:

Move to Phase 3.

### If 5 strategies fail:

Stop and present to the user:
- The 5 approaches attempted and why each failed
- The revised hypothesis about the bug's cause
- A suggestion to unblock

---

## Phase 3: Full Verification

1. Run the full test suite to detect regressions
2. If existing tests break:
   - Analyze whether it's a real regression or a flaky test
   - Fix real regressions
   - Report flaky tests without modifying them

---

## Phase 4: Summary

Present:
- **Cause**: the root cause in 1-2 sentences
- **Fix**: what was modified (files and lines)
- **Test**: the added test
- **Regressions**: none / list of detected issues

Do NOT commit automatically — wait for the user to ask.

---

## Notes

- Follow the project's testing conventions (see CLAUDE.md for specific helpers and constraints)
