# Claude Skills — Skills Réutilisables pour Claude Code

Collection de skills Claude Code génériques, extraites de projets en production et prêtes à l'emploi sur n'importe quel projet.

## Philosophie

Ces skills encodent des workflows de développement éprouvés : feature pipeline, bugfix test-first, revue de code, exploration d'approches, changelog métier. Elles sont conçues pour être **autonomes** (pas de référence à un projet spécifique) et **composables** (chaque skill fonctionne indépendamment).

Le repository inclut aussi un **template CLAUDE.md** avec des guidelines comportementales partagées, et un **hook de test automatique** qui lance les tests après chaque édition de fichier.

## Catalogue des skills

| Skill | Description | Argument |
|-------|------------|----------|
| `/feature` | Pipeline autonome : branche → implémentation → tests → commits → PR | description de la fonctionnalité |
| `/bugfix` | Diagnostic et correction test-first avec agents parallèles | description du symptôme |
| `/explore-approaches` | Compare 2 approches via agents en worktrees isolés | objectif à atteindre |
| `/polish` | Revue de code : bugs, conformité, couverture tests + corrections QCM | numéro de PR ou branche (optionnel) |
| `/changelog` | Génère le changelog métier à partir des commits | `full` pour tout régénérer (optionnel) |
| `/update-doc` | Détecte le drift code↔doc et propose les mises à jour | chemin ou nom du fichier doc (optionnel) |
| `/update-scenarios` | Intègre les nouvelles features dans les scénarios e2e par domaine | fonctionnalité ajoutée/modifiée |

Voir [SKILLS.md](SKILLS.md) pour la documentation complète avec exemples.

## Installation

### Installation rapide

```bash
# Tout installer dans votre projet
bin/install --all /chemin/vers/votre-projet

# Skills uniquement
bin/install --skills /chemin/vers/votre-projet

# Hook de test + settings.json
bin/install --hook /chemin/vers/votre-projet

# Template CLAUDE.md
bin/install --template /chemin/vers/votre-projet
```

### Installation interactive

```bash
bin/install /chemin/vers/votre-projet
```

Le script vous guide dans le choix des composants à installer.

### Installation manuelle

Copier les skills souhaitées dans `.claude/skills/` de votre projet :

```bash
cp -r skills/feature/.  /votre-projet/.claude/skills/feature/
cp -r skills/bugfix/.   /votre-projet/.claude/skills/bugfix/
# etc.
```

## Contenu du repository

```
claude-skills/
├── skills/                    # Skills prêtes à l'emploi
│   ├── bugfix/SKILL.md
│   ├── changelog/SKILL.md
│   ├── explore-approaches/SKILL.md
│   ├── feature/SKILL.md
│   ├── polish/SKILL.md
│   ├── update-doc/SKILL.md
│   └── update-scenarios/SKILL.md
├── templates/                 # Templates de configuration
│   ├── bin/demo               # Script démo enregistrable (ffmpeg, title cards, chapitres MKV)
│   ├── CLAUDE.md              # Guidelines comportementales
│   └── settings.json          # Hook PostToolUse
├── bin/                       # Scripts utilitaires
│   ├── install                # Installeur interactif
│   └── test-hook              # Hook auto-test générique
├── README.md
└── SKILLS.md                  # Documentation utilisateur des skills
```

## Créer une skill projet-spécifique

Les skills de ce repository sont génériques. Pour des besoins spécifiques à votre projet :

1. Créer le dossier `.claude/skills/ma-skill/` dans votre projet
2. Écrire un `SKILL.md` avec le frontmatter requis :
   ```yaml
   ---
   description: Description courte de la skill
   argument-hint: indice sur l'argument attendu
   ---
   ```
3. Structurer la skill en phases claires avec des points de contrôle

## Pré-requis

- **Claude Code** : les skills sont conçues pour Claude Code (cli)
- **Test runner** : les skills et templates utilisent `bin/rails test` par défaut (configurable via `TEST_CMD`)
- **jq** : requis par `bin/test-hook` pour parser le JSON d'entrée
- **gh** (optionnel) : requis par `/feature` et `/polish` pour créer des PRs
- **ffmpeg + ffprobe** (optionnel) : requis par `bin/demo --record` pour l'enregistrement vidéo
