---
description: 'Integrate new features into existing e2e scenarios to keep "ideal" user journeys up to date'
argument-hint: 'added/modified feature (e.g. "quality control at reception")'
---

# /update-scenarios — E2E Scenario Update

When a feature is added or modified, this skill integrates it into existing e2e demo scenarios so that user journeys stay up to date and recordable with `bin/demo --record`.

Each scenario corresponds to a **domain** or **focus** of the application workflow (e.g., reception, delivery, billing). They can be run together or in isolation.

**Feature**: $ARGUMENTS

**Prerequisite**: check the project's CLAUDE.md for test commands, code structure (routes, views, controllers) and framework conventions.

## Principles

- **Integration, not isolation**: the new feature fits into the existing workflow of the right domain, not into an orphan test
- **Ideal journey**: each scenario shows the typical user journey within a given domain
- **Multi-domain**: an app has multiple scenarios (one per domain/focus), runnable together (`bin/demo all`) or separately (`bin/demo reception`)
- **Video-readable**: human-readable, narrative test names — they appear as MKV chapters and in the bottom bar

---

## Phase 1: Understand the Feature and Its Domain

### 1A. Analyze the Feature

1. Identify the feature's files: routes, controllers/handlers, views/templates, JS
2. Trace the user journey: at what point in the workflow does this feature come in?
3. Identify interactions: forms, modals, selections, clicks, visual feedback

### 1B. Map Existing Scenarios

1. Read current demo scenarios:
   ```bash
   find test/ tests/ spec/ -name "demo_*" -o -name "*_workflow_*" -o -name "*_scenario_*" -o -name "*_e2e_*" 2>/dev/null
   ```
2. Read `bin/demo` (if it exists) to understand covered domains:
   - What targets exist? (e.g., `e2e`, `reception`, `delivery`)
   - What workflows does each target cover?
3. For each scenario, list the journey steps

### 1C. Identify the Target Domain

Determine **which scenario** the feature fits into:
- Feature related to reception → `reception` scenario
- Cross-cutting feature (e.g., notifications) → multiple scenarios impacted
- Feature for a new domain → new scenario to create

Then find the **insertion point** in the journey:

```
Existing "reception" workflow:    [Create] → [Physical Receipt] → [Validation] → [Close]
                                                                ↑
New feature:                                               [QC Check]
                                                                ↓
Updated workflow:                 [Create] → [Physical Receipt] → [QC Check] → [Validation] → [Close]
```

---

## Phase 2: Proposal

Present via `AskUserQuestion`:

- Which existing scenario(s) are impacted (domain)
- Where the feature fits in each journey
- Whether a new chapter (= test) is needed or it fits in an existing test
- Whether a new domain/scenario needs to be created

Options:
- "Validate integration" (Recommended)
- "Adjust"

**Splitting rule**: if the addition extends a test by more than ~20 seconds of video, extract a new test (= new MKV chapter).

**Wait for validation before modifying code.**

---

## Phase 3: Update Scenarios

### Modify an Existing Test

Insert the new steps at the right place in the journey:

```ruby
test "Complete reception with validation" do
  # ... existing creation steps ...
  demo_pause

  # ── NEW: Quality control ──
  click_on "Start inspection"
  demo_pause

  within_modal do
    select "Conforming", from: "Result"
    demo_tick
    click_on "Validate"
  end
  demo_pause

  assert_text "Inspection validated"
  demo_pause
  # ── END NEW ──

  # ... existing closing steps ...
end
```

### Add a New Chapter (test) in an Existing Scenario

If the feature warrants its own video chapter:

```ruby
test "Quality control — anomaly reported and resolved" do
  reception = setup_reception_pending_qc

  sign_in_as("qc_manager")
  demo_pause

  visit reception_path(reception)
  demo_pause

  # ... QC interactions ...
  demo_pause(4)  # show the result
end
```

### Create a New Scenario (new domain)

If the feature opens a new application domain:

1. Create the e2e test file for the domain
2. Design the complete journey for the domain (not just the feature)
3. Add to `bin/demo`: `target_file`, `target_title`, `expand_targets`

### Test Names: Human-Readable and Narrative

Names appear in the video. Write them like chapter titles:

```ruby
# ✗ Technical
test "reception_with_qc_check_and_anomaly_report"

# ✓ Narrative — this is a video chapter title
test "Quality control — anomaly reported and resolved"
```

### Interaction Tempo

| Action | Pause | Helper |
|--------|-------|--------|
| Page load, form submission | 2-3s | `demo_pause` |
| Click, focus, input | 0.3-0.5s | `demo_tick` |
| Important result to show | 3-5s | `demo_pause(4)` |

### Test Data

- Adapt the existing scenario `setup` to include the feature's data
- If data is complex, add a private helper `setup_<context>_data`
- Realistic names: `"Martin Transport"` not `"Test 1"` — the UI should look like production

---

## Phase 4: Integration with bin/demo

### If a new scenario (domain) is created:

1. Add to `target_file()`: mapping target → test file
2. Add to `target_title()`: human-readable title for title cards
3. Include in `expand_targets()` (composite `all`)
4. Update `--list`

### If only existing tests are modified:

Nothing to change in `bin/demo` — MKV chapters update automatically from test names.

### If `bin/demo` doesn't exist:

Propose installing the template from claude-skills.

---

## Phase 5: Verification

1. Run the modified scenario (project's test command)

2. Visually verify if possible:
   ```bash
   bin/demo <target>
   ```

3. Check journey consistency: do the steps flow naturally?

---

## Phase 6: Summary

Present:
- **Impacted domain(s)**: which scenarios were modified
- **Insertion point**: where the feature fits in the workflow
- **Chapters added/modified**: test names (= video titles)
- **New domain**: if a scenario was created, its `bin/demo` target

Do NOT commit automatically — wait for the user to ask.

---

## Notes

- The goal is to maintain **ideal journeys per domain** — not a collection of isolated tests
- Each domain = one scenario = one `bin/demo` target = one video with chapters
- If an interaction helper is missing (drag-drop, right-click, canvas), report it rather than inventing one
- Demo scenarios complement functional tests — they don't replace them
