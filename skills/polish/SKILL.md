---
description: Polish PRs and uncommitted code — review, propose fixes as QCM, apply only approved fixes, re-review, report
argument-hint: PR number or branch (optional — defaults to all open PRs + uncommitted changes)
---

# /polish — PR Quality Polish

Polish code that isn't on main yet: review it, propose fixes, apply only user-approved changes, and give objective feedback.

**Target**: $ARGUMENTS

## Important Principles

- **Non-intrusive**: NEVER modify code without explicit user approval
- **QCM-driven**: Present all issues as a selectable checklist — user picks what to fix
- **Objective**: Give honest, unbiased feedback — don't sugar-coat
- **Surgical**: Fix only what was selected, nothing else
- **Test-driven**: Run tests after every fix to catch regressions

---

## Phase 1: Identify Targets

### If `$ARGUMENTS` is provided:
- If it's a number → treat as PR number: `gh pr view <number> --json number,title,state,headRefName,url`
- If it's a branch name → find its PR: `gh pr list --head <branch> --json number,title,state,url`

### If `$ARGUMENTS` is empty (default — review everything not on main):

1. **Uncommitted changes** on current branch:
   ```bash
   git status
   git diff HEAD
   ```

2. **All open PRs** targeting main:
   ```bash
   gh pr list --state open --base main --json number,title,headRefName,url
   ```

3. Present a summary of what will be reviewed and ask user to confirm scope.

### Processing multiple PRs:
Process each PR sequentially. For each PR:
- Checkout the branch: `git checkout <branch>`
- Run Phases 2–6
- Return to main before the next PR: `git checkout main`
- Present a combined report at the end (Phase 7)

**Output**: List of targets (PRs and/or uncommitted changes) with titles and URLs.

---

## Phase 2: Code Review

For each target, get the diff:
- PR: `gh pr diff <number>`
- Uncommitted: `git diff main...HEAD` + `git diff`

Launch 3 parallel review agents (use Task tool with sonnet model):

1. **Bug scanner**: Read the diff. Scan for bugs, logic errors, nil references, missing error handling, race conditions, edge cases. Focus on real bugs, not style. Return issues with file:line references and severity (critical/important/minor).

2. **CLAUDE.md compliance**: Read the project CLAUDE.md and the diff. Check for violations of project conventions and guidelines. Flag only clear, specific violations with the exact CLAUDE.md rule being broken. Return issues with references.

3. **Test & doc coverage**: Check if modified code has adequate test coverage. Look for: missing test files, untested new methods, broken assertions, outdated comments/docs, missing i18n keys. Return issues with references.

Each agent must return a structured list:
```
- [severity] file:line — description (reason: ...)
```

---

## Phase 3: Consolidate & Deduplicate

Merge results from all 3 agents. Remove duplicates. Group by severity:
- **Critical**: Bugs that will break in production
- **Important**: CLAUDE.md violations, missing tests, logic issues
- **Minor**: Style, documentation, nitpicks

---

## Phase 4: QCM — User Selects Fixes

**CRITICAL: DO NOT SKIP THIS PHASE.**

Present all issues as an `AskUserQuestion` with `multiSelect: true`. Group by severity.

Format each option as:
```
label: "[Critical] file.rb:42 — nil reference on stop.address"
description: "stop.address can be nil since PR #153 removed the NOT NULL constraint"
```

If there are more than 4 issues, present the top 4 most important with AskUserQuestion and list the rest as text. The user can type "other" to select from the remaining issues.

Include a "Fix all critical + important" convenience option.
Include a "Skip all — just report" option.

**Wait for the user's selection before proceeding.**

If the user selects "Skip all", jump to Phase 7 (Report).

---

## Phase 5: Apply Selected Fixes

For each selected issue, in order:

1. Read the target file
2. Make the minimal fix (surgical, no adjacent changes)
3. If tests are affected, update them too

After all fixes, run the modified test files.

If tests fail:
- Analyze the failure
- Fix and retry (max 2 attempts per file)
- If still failing after 2 attempts, revert the fix and note it in the report

---

## Phase 6: Re-Review (Light Pass)

Launch a single review agent to verify:
- All selected issues are actually resolved
- No new issues were introduced by the fixes
- Tests pass

Present results to the user. If new issues found, offer to fix them (same QCM approach).

---

## Phase 7: Report & Feedback

### 7A. Objective Quality Assessment

For each PR reviewed, give an honest assessment on a 5-point scale:

| Score | Meaning |
|-------|---------|
| 5/5   | Production-ready, excellent code |
| 4/5   | Good, minor issues only |
| 3/5   | Acceptable, some important issues |
| 2/5   | Needs work, critical issues remain |
| 1/5   | Not ready, fundamental problems |

Be objective. Don't inflate scores.

### 7B. Commit & Push (if fixes were made)

Ask the user whether to commit the fixes. If yes:
- Stage modified files
- Commit with message: `polish: fix N issues from code review`
- Push to the PR branch

### 7C. PR Comment (optional)

Ask the user whether to post a summary comment on each PR. If yes, use this format:

```
### Polish Report

**Quality**: X/5
**Review**: Found N issues (X critical, Y important, Z minor)
**Fixed**: X issues fixed, Y skipped

<details>
<summary>Details</summary>

| Status | Severity | File | Issue |
|--------|----------|------|-------|
| Fixed | Critical | file.rb:42 | description |
| Skipped | Minor | file.rb:10 | description |

</details>

Generated with [Claude Code](https://claude.ai/code) — `/polish`
```

---

## Notes

- If a PR has no issues, just say it passes. Don't invent problems, don't praise either — be factual.
- Never commit without asking. Never push without asking.
- Keep the conversation flowing — the user should feel in control at every step.
- For multi-PR reviews, give a combined summary at the end with overall readiness assessment.
