# Claude Skills — Reusable Skills for Claude Code

Collection of generic Claude Code skills, extracted from production projects and ready to use on any project.

## Philosophy

These skills encode proven development workflows: feature pipeline, test-first bugfix, code review, approach exploration, business changelog. They are designed to be **standalone** (no reference to a specific project) and **composable** (each skill works independently).

The repository also includes a **CLAUDE.md template** with shared behavioral guidelines, and an **automatic test hook** that runs tests after each file edit.

## Skill Catalog

| Skill | Description | Argument |
|-------|------------|----------|
| [`/feature`](skills/feature/SKILL.md) | Autonomous pipeline: branch, implement, test, commit, PR | feature description |
| [`/bugfix`](skills/bugfix/SKILL.md) | Test-first diagnosis and fix with parallel agents | symptom description |
| [`/explore-approaches`](skills/explore-approaches/SKILL.md) | Compare 2 approaches via agents in isolated worktrees | goal to achieve |
| [`/polish`](skills/polish/SKILL.md) | Code review: bugs, compliance, test coverage + QCM fixes | PR number or branch (optional) |
| [`/changelog`](skills/changelog/SKILL.md) | Generate business changelog from commits | `full` to regenerate everything (optional) |
| [`/update-doc`](skills/update-doc/SKILL.md) | Detect code-doc drift and propose updates | doc file path or name (optional) |
| [`/update-scenarios`](skills/update-scenarios/SKILL.md) | Integrate new features into e2e scenarios by domain | added/modified feature |

See [SKILLS.md](SKILLS.md) for the full documentation with examples.

## Installation

### Quick Install

```bash
# Install everything into your project
bin/install --all /path/to/your-project

# Skills only
bin/install --skills /path/to/your-project

# Test hook + settings.json
bin/install --hook /path/to/your-project

# CLAUDE.md template
bin/install --template /path/to/your-project
```

### Interactive Install

```bash
bin/install /path/to/your-project
```

The script guides you through choosing which components to install.

### Manual Install

Copy the desired skills into your project's `.claude/skills/`:

```bash
cp -r skills/feature/.  /your-project/.claude/skills/feature/
cp -r skills/bugfix/.   /your-project/.claude/skills/bugfix/
# etc.
```

## Repository Contents

```
claude-skills/
├── skills/                    # Ready-to-use skills
│   ├── bugfix/SKILL.md
│   ├── changelog/SKILL.md
│   ├── explore-approaches/SKILL.md
│   ├── feature/SKILL.md
│   ├── polish/SKILL.md
│   ├── update-doc/SKILL.md
│   └── update-scenarios/SKILL.md
├── templates/                 # Configuration templates
│   ├── bin/demo               # Recordable demo script (copied to your project's bin/demo)
│   ├── CLAUDE.md              # Behavioral guidelines
│   └── settings.json          # PostToolUse hook
├── bin/                       # Utility scripts
│   ├── install                # Interactive installer
│   └── test-hook              # Generic auto-test hook
├── README.md
└── SKILLS.md                  # User-facing skill documentation
```

## Creating a Project-Specific Skill

The skills in this repository are generic. For project-specific needs:

1. Create the folder `.claude/skills/my-skill/` in your project
2. Write a `SKILL.md` with the required frontmatter:
   ```yaml
   ---
   description: Short skill description
   argument-hint: hint about the expected argument
   ---
   ```
3. Structure the skill in clear phases with checkpoints

## Prerequisites

- **Claude Code**: these skills are designed for Claude Code (CLI)
- **Test runner**: skills and templates default to `bin/rails test` (configurable via `TEST_CMD`)
- **jq**: required by `bin/test-hook` to parse JSON input
- **gh** (optional): required by `/feature` and `/polish` for creating PRs
- **ffmpeg + ffprobe** (optional): required by [`templates/bin/demo`](templates/bin/demo) `--record` for video recording
