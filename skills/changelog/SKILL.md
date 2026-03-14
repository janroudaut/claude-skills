---
description: Generate or update the business changelog (CHANGELOG.md) from recent commits
argument-hint: '"full" to regenerate everything (optional, defaults to adding a new milestone)'
---

# /changelog — Business Changelog Update

Generates business-oriented changelog entries (`CHANGELOG.md`) from Git history.

**Arguments**: $ARGUMENTS

---

## Mode: Determine Execution Mode

- If `$ARGUMENTS` contains `full` → **Full mode**: regenerate the entire file
- Otherwise → **Incremental mode**: add a new milestone

---

## Incremental Mode (default)

### Step 1: Read Current State

1. Read `CHANGELOG.md` to identify the last milestone (e.g., `v3`)
2. Identify the date of the last milestone by browsing the corresponding commits

### Step 2: Fetch Recent Commits

```bash
git log --oneline --no-merges main
```

Browse commits since the last known milestone. If no significant commits are found, inform the user and stop.

### Step 3: Analyze and Group

For each relevant commit, read the diff to understand the change:

```bash
git show --stat <sha>
git show <sha> -- <key_files>
```

Ignore:
- Merge commits
- Dependency updates (dependabot)
- Purely technical changes (CI, config, internal refactoring) unless significant
- Test-only modifications

Group changes into business categories (New Features, Quality, Infrastructure).

### Step 4: Propose the New Milestone

Determine the version number (last + 1) and propose a milestone title.

Generate the block in this format:

```markdown
## vX — Milestone Title (period)

Short milestone description (1-2 sentences).

**New Features:**
- User-oriented item, not technical
- Each item describes the benefit, not the implementation
```

### Step 5: User Validation

Present the generated block to the user with `AskUserQuestion`:
- Option 1: "Validate and write" — insert at the top of the file (after the `# Changelog` heading)
- Option 2: "Change the milestone title" — the user proposes a different title
- Option 3: "Cancel"

### Step 6: Write

If validated, insert the new milestone at the top of `CHANGELOG.md`, right after the `# Changelog` line, followed by a blank line and the `---` separator.

---

## Full Mode (`full`)

### Step 1: Fetch Full History

```bash
git log --oneline --no-merges --reverse main
```

### Step 2: Identify Milestones

Group commits into coherent business milestones. Splitting criteria:
- Major change in functional domain
- Significant pause between development periods
- Thematic consistency of commits

### Step 3: Generate the Complete File

For each milestone, generate a block in the standard format. Number sequentially (v1, v2, v3...).

### Step 4: User Validation

Present the complete file with `AskUserQuestion`:
- Option 1: "Validate and write"
- Option 2: "Cancel"

### Step 5: Write

If validated, rewrite `CHANGELOG.md` entirely.

---

## Writing Rules

- **Business-oriented**: describe what the user can do, not how it's implemented
- **Concise**: each item in one sentence, no unnecessary technical jargon
- **No file/class names**: say "automatic document analysis" not "OcrLlmService"
- **No PR/commit numbers** in the final changelog

---

## Notes

- Never write without user validation
- If the number of commits is very large (> 100), ask the user to specify a date range or commit count
- The file uses the heading `# Changelog`
