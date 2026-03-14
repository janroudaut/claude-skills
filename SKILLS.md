# Claude Code Skills

Commands available in Claude Code sessions via `/name`.

## Development Workflows

### [`/feature`](skills/feature/SKILL.md) `<description>` — Feature → PR Pipeline
Autonomous pipeline: branch, implement, test, commit, PR.
- **Argument**: description of the feature to implement (free text)
- Scope validation before coding
- Separate commits by type (code, tests, migrations)
- Draft PR if unresolved blocker after 5 attempts

Example: `/feature Add a date filtering system on the orders list`

### [`/bugfix`](skills/bugfix/SKILL.md) `<symptom>` — Test-First Fix
Autonomous diagnosis and fix with parallel agents.
- **Argument**: description of the observed symptom (free text)
- Agent A: diagnosis (traces the flow, identifies root cause)
- Agent B: reproduction test (writes and verifies the test)
- Fix loop: up to 5 different strategies without asking
- Verification: full test suite after the fix

Example: `/bugfix Invoice total doesn't update after removing a line item`

### [`/explore-approaches`](skills/explore-approaches/SKILL.md) `<goal>` — Parallel Exploration
Compare 2 implementation approaches via agents in isolated worktrees.
- **Argument**: description of the feature or problem to solve (free text)
- Each agent produces working code + tests
- Comparison on: modified files, complexity, consistency, tests
- User chooses which approach to apply

Example: `/explore-approaches Real-time notification system for status updates`

## Quality and Documentation

### [`/polish`](skills/polish/SKILL.md) `[PR|branch]` — Code Review
Review PRs and uncommitted code: bugs, CLAUDE.md compliance, test coverage.
- **Argument** (optional): PR number or branch name. Without argument: reviews all open PRs + uncommitted changes
- 3 parallel review agents
- QCM presentation of found issues
- Surgical application of selected fixes
- Quality score 1-5

Example: `/polish 42` or `/polish feat/notifications` or `/polish` (all)

### [`/update-doc`](skills/update-doc/SKILL.md) `[file]` — Documentation Update
Detects gaps between code and documentation, proposes and applies updates.
- **Argument** (optional): doc file path or name. Without argument: scans the project and offers a choice
- Automatic discovery of project Markdown files
- Inference of code related to each doc
- Drift detection: outdated info, undocumented features, inconsistencies
- QCM proposal of changes, selective application

Example: `/update-doc README.md` or `/update-doc docs/api.md` or `/update-doc` (full scan)

### [`/update-scenarios`](skills/update-scenarios/SKILL.md) `<feature>` — E2E Scenario Update
Integrates new features into existing e2e demo scenarios, by domain.
- **Argument**: added or modified feature (free text)
- Maps application workflows (routes, controllers, views) to design the user journey
- Identifies target domain/scenario and insertion point in the existing journey
- Narrative, human-readable test names (displayed in the video as chapters)
- Integration with `bin/demo` (targets, title cards, MKV chapters)

Example: `/update-scenarios quality control at reception` or `/update-scenarios notification system`

### [`/changelog`](skills/changelog/SKILL.md) `[full]` — Changelog
Generates or updates `CHANGELOG.md` from recent commits.
- **Argument** (optional): `full` to regenerate everything. Without argument: adds a new milestone

Example: `/changelog` or `/changelog full`

## Automatic Hook

The `bin/test-hook` hook runs tests automatically after each file edit:
- Non-code files (.md, .json, etc.) → ignored
- Backend files → unit/integration tests
- Frontend files (views, JS, CSS) → unit + system tests
