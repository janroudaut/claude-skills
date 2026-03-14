---
description: Detect drift between code and documentation, propose and apply necessary updates
argument-hint: doc file path or name to update (optional — without argument, scans the project)
---

# /update-doc — Documentation Update

Analyzes gaps between the current code and documentation, then proposes and applies necessary updates.

**Scope**: $ARGUMENTS

**Prerequisite**: check the project's CLAUDE.md for code structure and conventions.

## Principles

- **Non-intrusive**: never modify without user validation
- **Style-faithful**: respect the structure and tone of existing docs
- **Discovery-based**: no hardcoded scopes — scan the project to find docs
- **Reader-oriented**: adapt the level of detail to the document's target audience

---

## Phase 1: Discovery and Scope Selection

### If `$ARGUMENTS` is provided:
- If it's a file path → target that file directly
- If it's a keyword → search for matching doc files (`README`, `api`, `setup`, etc.)

### If `$ARGUMENTS` is empty:

1. Scan the project to find documentation files:
   ```bash
   find . -name "*.md" -not -path "./.claude/*" -not -path "./vendor/*" -not -path "./node_modules/*" | head -30
   ```

2. Present the found files via `AskUserQuestion` with `multiSelect: true`:
   - Group by folder if many files
   - "Scan all" option for a complete analysis

**Wait for selection before continuing.**

---

## Phase 2: Gap Analysis

For each selected doc file:

### 2A. Read Existing Content

Read the full Markdown file.

### 2B. Identify Related Code

Infer the documented source files by searching for:
- File paths mentioned in the doc
- Referenced classes, methods, commands
- Cited file/folder names

Explore the corresponding code to understand the current state.

### 2C. Detect Gaps

For each section of the doc, identify:
- **Outdated information**: commands, paths, options, names that have changed in the code
- **Undocumented features**: present in the code but missing from the doc
- **Removed features**: documented but deleted from the code
- **Inconsistencies**: versions, dependencies, examples that no longer match

---

## Phase 3: Propose Changes

### 3A. Gap Summary

Present a clear text summary of all gaps found, organized by file.

### 3B. Select Changes

Use `AskUserQuestion` with `multiSelect: true` to list proposed changes.

Format each option:
```
label: "file.md — Short change description"
description: "Detail: what is outdated / missing and what will be modified"
```

Include options:
- **"Apply all"**: apply all proposed changes
- **"Cancel"**: modify nothing

If more than 4 changes, present the top 4 most important via `AskUserQuestion` and list the rest as text. The user can reply "Other" to select from the remaining ones.

**Wait for the user's selection before continuing.**

If the user chooses "Cancel", stop with a confirmation message.

---

## Phase 4: Apply Changes

For each selected change:
1. Read the target file
2. Apply the change (Edit or Write depending on scope)
3. Respect existing style: headings, lists, formatting, level of detail, language

---

## Phase 5: Verification

### 5A. Proofreading

Re-read each modified file to verify:
- Content consistency
- No typos or formatting issues
- Correct Markdown formatting

### 5B. Link Verification

Verify that:
- Links to files or images point to existing resources
- Internal anchors (`[...](#section)`) are valid
- Cross-references between doc files are correct

### 5C. Final Summary

Present to the user:
- The list of modified files
- A summary of each applied change
- Any unresolved issues

Do NOT commit automatically — wait for the user to ask.

---

## Notes

- If no gaps are found, simply say so — don't invent changes
- Never write implementation details in docs intended for end users
- Respect the language of the existing document (don't translate an EN doc to another language or vice versa)
