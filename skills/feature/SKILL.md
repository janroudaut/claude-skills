---
description: Autonomous pipeline — feature branch, implement, test, commit, PR
argument-hint: description of the feature to implement
---

# /feature — Feature → PR Pipeline

Implements a feature end-to-end: branch, code, tests, commit, PR. Autonomous pipeline with checkpoints.

**Feature**: $ARGUMENTS

**Prerequisite**: check the project's CLAUDE.md for test commands and framework conventions.

## Principles

- **Autonomous**: move forward without asking unless truly blocked
- **Incremental**: separate logical commits, not one monolithic commit
- **Verified**: each step validated before moving to the next
- **Reversible**: dedicated branch, main never touched

---

## Phase 1: Quick Scoping (2-3 minutes max)

1. Read relevant files to understand the context
2. Check existing similar implementations in the codebase
3. Define the scope in 3-5 bullet points:
   ```
   - [ ] Goal 1
   - [ ] Goal 2
   - [ ] Goal 3
   ```

Present the scope to the user via `AskUserQuestion`:
- "Validate scope" (Recommended)
- "Adjust scope"

**Do NOT start coding before scope validation.**

---

## Phase 2: Branch and Implementation

1. Create the branch:
   ```bash
   git checkout -b feat/<kebab-case-name>
   ```

2. Implement the feature following the project conventions:
   - Respect existing patterns in the codebase
   - Follow conventions described in the project's CLAUDE.md

3. After each logical block of changes, run the relevant tests

4. Fix failures before continuing.

---

## Phase 3: Tests

1. Write tests for the new feature:
   - E2e/system test if it's a UI change
   - Unit/integration test for business logic
   - Cover the happy path + one edge case

2. Run the new tests

3. If failure, fix and re-run (max 5 iterations).

4. Run the full suite to detect regressions

5. **If persistent blocker** (tests not passing after 5 attempts):
   - Do NOT stop — continue the pipeline in degraded mode
   - Document the issue for the PR (see Phase 5)

---

## Phase 4: Commits

Separate changes into logical commits:
- Application code (models, controllers, views, JS, CSS)
- Tests
- Migrations (if applicable)

Use descriptive commit messages.
Do NOT use conventional prefixes except `[meta]` and `[regen]`.

---

## Phase 5: PR

1. Push the branch:
   ```bash
   git push -u origin feat/<branch-name>
   ```

2. Create the PR:
   - **If all tests pass** → normal PR:
     ```bash
     gh pr create --title "<short title>" --body "$(cat <<'EOF'
     ## Summary
     <3-5 bullet points describing the changes>

     ## Tests
     - [x] Unit/integration tests added
     - [x] System tests added (if UI)
     - [x] Full suite green

     Generated with [Claude Code](https://claude.com/claude-code) — `/feature`
     EOF
     )"
     ```
   - **If a blocker persists** → draft PR with documented issue:
     ```bash
     gh pr create --draft --title "<short title>" --body "$(cat <<'EOF'
     ## Summary
     <3-5 bullet points describing the changes>

     ## Unresolved Blocker
     <problem description, approaches tried, suggestion to unblock>

     ## Tests
     - [x] Unit/integration tests added
     - [ ] Full suite green — blocker above

     Generated with [Claude Code](https://claude.com/claude-code) — `/feature`
     EOF
     )"
     ```

3. Display the PR URL.

---

## Phase 6: Summary

Present:
- **PR**: URL
- **Modified files**: list
- **Tests added**: count and coverage
- **Scope covered**: reminder of checked goals

Ask the user if they want to:
- Run `/polish` on the PR
- Merge directly
- Adjust something

---

## Notes

- NEVER modify `CLAUDE.md`, `MEMORY.md`, or `.claude/` files
- If a scope goal turns out to be too complex, flag it and propose splitting it into a separate PR
- Follow project-specific constraints described in the project's CLAUDE.md
