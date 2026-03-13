# Skills Claude Code

Commandes disponibles en session Claude Code via `/nom`.

## Workflows de développement

### `/feature <description>` — Pipeline feature → PR
Pipeline autonome : branche → implémentation → tests → commits → PR.
- **Argument** : description de la fonctionnalité à implémenter (texte libre)
- Validation du scope avant de coder
- Commits séparés par type (code, tests, migrations)
- PR en draft si blocage non résolu après 5 tentatives

Exemple : `/feature Ajouter un système de filtrage par date sur la liste des commandes`

### `/bugfix <symptôme>` — Correction test-first
Diagnostic et correction autonome avec agents parallèles.
- **Argument** : description du symptôme observé (texte libre)
- Agent A : diagnostic (trace le flux, identifie la cause)
- Agent B : test de reproduction (écrit et vérifie le test)
- Boucle de correction : jusqu'à 5 stratégies différentes sans demander
- Vérification : suite complète après le fix

Exemple : `/bugfix Le total de la facture ne se met pas à jour après suppression d'une ligne`

### `/explore-approaches <objectif>` — Exploration parallèle
Compare 2 approches d'implémentation via des agents en worktrees isolés.
- **Argument** : description de la fonctionnalité ou du problème à résoudre (texte libre)
- Chaque agent produit du code fonctionnel + tests
- Comparaison sur : fichiers modifiés, complexité, cohérence, tests
- L'utilisateur choisit l'approche à appliquer

Exemple : `/explore-approaches Système de notifications en temps réel pour les mises à jour de statut`

## Qualité et documentation

### `/polish [PR|branche]` — Revue de code
Revue de PRs et code non commité : bugs, conformité CLAUDE.md, couverture tests.
- **Argument** (optionnel) : numéro de PR ou nom de branche. Sans argument : revue de toutes les PRs ouvertes + changements non commités
- 3 agents de revue parallèles
- Présentation QCM des issues trouvées
- Application chirurgicale des corrections sélectionnées
- Score qualité 1-5

Exemple : `/polish 42` ou `/polish feat/notifications` ou `/polish` (tout)

### `/update-doc [fichier]` — Mise à jour documentation
Détecte les écarts entre le code et la documentation, propose et applique les mises à jour.
- **Argument** (optionnel) : chemin ou nom du fichier doc. Sans argument : scanne le projet et propose un choix
- Découverte automatique des fichiers Markdown du projet
- Inférence du code lié à chaque doc
- Détection du drift : infos obsolètes, fonctionnalités non documentées, incohérences
- Proposition QCM des modifications, application sélective

Exemple : `/update-doc README.md` ou `/update-doc docs/api.md` ou `/update-doc` (scan complet)

### `/update-scenarios <fonctionnalité>` — Mise à jour des scénarios e2e
Intègre les nouvelles fonctionnalités dans les scénarios de démo e2e existants, par domaine.
- **Argument** : fonctionnalité ajoutée ou modifiée (texte libre)
- Cartographie les workflows Rails (routes, contrôleurs, vues) pour concevoir le parcours utilisateur
- Identifie le domaine/scénario cible et le point d'insertion dans le parcours existant
- Noms de tests narratifs et humains (affichés dans la vidéo comme chapitres)
- Intégration avec `bin/demo` (targets, title cards, MKV chapitres)

Exemple : `/update-scenarios contrôle qualité à la réception` ou `/update-scenarios système de notifications`

### `/changelog [full]` — Changelog
Génère ou met à jour `CHANGES_fr.md` à partir des commits récents.
- **Argument** (optionnel) : `full` pour tout régénérer. Sans argument : ajoute un nouveau jalon

Exemple : `/changelog` ou `/changelog full`

## Hook automatique

Le hook `bin/test-hook` lance les tests automatiquement après chaque édition de fichier :
- Fichiers non-code (.md, .json, etc.) → ignorés
- Fichiers backend → tests unitaires/intégration
- Fichiers frontend (vues, JS, CSS) → tests unitaires + système
