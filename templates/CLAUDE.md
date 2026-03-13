# CLAUDE.md

Rules and conventions for Claude Code on this repository. For acquired knowledge (architecture, project state, legacy references), see the project memory file.

<!-- PROJECT-SPECIFIC: Remplacer cette section par l'identité de votre projet -->
## Project Identity

<!-- Décrire le projet : stack technique, domaine métier, frameworks utilisés -->
<!-- Exemple :
[Framework] application. Handles [domaine métier].

**Stack:** [frameworks, libraries, outils]
-->

<!-- PROJECT-SPECIFIC: Adapter les commandes à votre projet -->
## Commands

<!-- Lister les commandes de développement, test, lint, etc.
```bash
# Development
<commande serveur>               # Start dev server
<commande setup>                 # Setup DB / dependencies

# Testing
<commande tests>                 # Run all tests
<commande test fichier>          # Run a single test file

# Linting & Security
<commande lint>                  # Lint with auto-correct
```
-->

---

## Behavioral Guidelines

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

### 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.
- Before planning a new feature, quickly check if it's already implemented.
- **No band-aids:** never work around a problem. Always fix the root cause.
- **Full-stack debugging:** trace the full data flow (save → load → render) from the start. Don't spend too long on one side (client vs server) before checking the other.

### 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.
- When enhancing existing UI, **modify** the existing code — don't replace it with a new design.
- Don't create new models, services, or abstractions unless explicitly asked. Prefer ephemeral/inline solutions.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

### 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
```

When modifying views or user workflows significantly, update the corresponding tests in the same change.

Don't enter extended exploration/planning phases without producing code. If the task is clear, start implementing within the first 2-3 messages. Ask before spending time on plans.

- **Incremental delivery:** break large features into small, working increments. Deliver phase 1 immediately rather than proposing a massive plan. Ask before expanding scope.
- **Bug fixes:** always write a failing test BEFORE attempting a fix. If the first fix doesn't work, try a fundamentally different strategy — not a minor variation of the same approach. Use `/bugfix` for the full workflow.
- **Verify fixes:** run the relevant tests after every fix to confirm it actually works before reporting success.

<!-- PROJECT-SPECIFIC: Ajouter les conventions UI/UX spécifiques à votre projet -->
### 5. UI/UX

<!-- Exemples de conventions UI/UX :
- Palette CSS, système d'icônes
- Frameworks UI (Bootstrap, Tailwind, etc.)
- Conventions JS (Stimulus, vanilla, etc.)
- Règles de responsive/mobile
-->

<!-- PROJECT-SPECIFIC: Ajouter les conventions de framework spécifiques à votre projet -->
### 6. Framework Conventions

<!-- Exemples :
- Conventions de templates (ERB, Jinja, JSX, etc.)
- Asset pipeline / bundler
- Background jobs
- i18n
-->

### 7. Git

- When the user says a change is ready to commit, just commit it. Don't investigate further or suggest additional changes unless asked.
- **Commit messages:** Describe the result (what the code does now), not the journey (what was replaced/corrected along the way). Don't mention intermediate steps.
- Only stage files directly related to the task. Never include `.claude/`, skill files, config files, or unrelated changes in a commit.
- Don't bundle unrelated fixes or refactors into a commit — one concern per commit.
- **Never push without explicit request.** Commit = OK when asked. Push = always ask first.
- **Special commit prefixes:**
  - `[meta]`: Claude files and memory (`CLAUDE.md`, `.claude/skills/`, `.claude/*memory*`)
  - `[regen]`: files generated by `/changelog` or `/update-doc` skills (`CHANGES_fr.md`, `doc/*.md`)

<!-- PROJECT-SPECIFIC: Ajouter les conventions de test spécifiques à votre projet -->
### 8. Testing

<!-- Exemples :
- Framework de mock/stub utilisé
- Helpers de test spécifiques
- Règles de parallélisation
- Conventions pour les tests système
-->

### 9. Environment

- Commands run inside WSL (the IDE is on a Windows host).

<!-- PROJECT-SPECIFIC: Ajouter les contraintes d'environnement spécifiques -->

### 10. Language & Accents

- All French text (docs, comments, commit messages, UI strings) MUST use proper accents: é, è, ê, à, ù, î, ô, ç, etc.
- Never write "e" where "é/è/ê" is needed — e.g., "préparé", "référence", "épisode", "clé", "déploiement".

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.
