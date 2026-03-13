---
description: Pipeline autonome feature branch → implémentation → tests → commit → PR
argument-hint: description de la fonctionnalité à implémenter
---

# /feature — Pipeline Feature → PR

Implémente une fonctionnalité de bout en bout : branche, code, tests, commit, PR. Pipeline autonome avec points de contrôle.

**Fonctionnalité**: $ARGUMENTS

## Principes

- **Autonome** : avancer sans demander sauf blocage réel
- **Incrémental** : commits logiques séparés, pas un gros commit monolithique
- **Vérifié** : chaque étape validée avant de passer à la suivante
- **Réversible** : branche dédiée, main jamais touché

---

## Phase 1 : Cadrage rapide (2-3 minutes max)

1. Lire les fichiers pertinents pour comprendre le contexte
2. Consulter les implémentations existantes similaires dans le codebase
3. Définir le scope en 3-5 bullet points :
   ```
   - [ ] Objectif 1
   - [ ] Objectif 2
   - [ ] Objectif 3
   ```

Présenter le scope à l'utilisateur via `AskUserQuestion` :
- "Valider le scope" (Recommandé)
- "Ajuster le scope"

**Ne PAS commencer à coder avant validation du scope.**

---

## Phase 2 : Branche et implémentation

1. Créer la branche :
   ```bash
   git checkout -b feat/<kebab-case-name>
   ```

2. Implémenter la fonctionnalité en suivant les conventions du projet :
   - Respecter les patterns existants dans le codebase
   - Suivre les conventions décrites dans le CLAUDE.md du projet

3. Après chaque bloc logique de changements, lancer les tests pertinents

4. Corriger les échecs avant de continuer.

---

## Phase 3 : Tests

1. Écrire les tests pour la nouvelle fonctionnalité :
   - Test e2e/système si c'est un changement UI
   - Test unitaire/intégration pour la logique métier
   - Couvrir le happy path + un edge case

2. Lancer les nouveaux tests

3. Si échec, corriger et relancer (max 5 itérations).

4. Lancer la suite complète pour détecter les régressions

5. **Si blocage persistant** (tests qui ne passent pas après 5 tentatives) :
   - Ne PAS s'arrêter — continuer le pipeline en mode dégradé
   - Documenter le problème pour la PR (voir Phase 5)

---

## Phase 4 : Commits

Séparer les changements en commits logiques :
- Code applicatif (modèles, contrôleurs, vues, JS, CSS)
- Tests
- Migrations (si applicable)

Utiliser des messages descriptifs en français avec accents corrects.
Ne PAS utiliser de préfixes conventionnels sauf `[meta]` et `[regen]`.

---

## Phase 5 : PR

1. Pousser la branche :
   ```bash
   git push -u origin feat/<branch-name>
   ```

2. Créer la PR :
   - **Si tous les tests passent** → PR normale :
     ```bash
     gh pr create --title "<titre court>" --body "$(cat <<'EOF'
     ## Résumé
     <3-5 bullet points décrivant les changements>

     ## Tests
     - [x] Tests unitaires/intégration ajoutés
     - [x] Tests système ajoutés (si UI)
     - [x] Suite complète verte

     🤖 Generated with [Claude Code](https://claude.com/claude-code) — `/feature`
     EOF
     )"
     ```
   - **Si un blocage persiste** → PR en draft avec le problème documenté :
     ```bash
     gh pr create --draft --title "<titre court>" --body "$(cat <<'EOF'
     ## Résumé
     <3-5 bullet points décrivant les changements>

     ## ⚠️ Blocage non résolu
     <description du problème, approches tentées, suggestion pour débloquer>

     ## Tests
     - [x] Tests unitaires/intégration ajoutés
     - [ ] Suite complète verte — blocage ci-dessus

     🤖 Generated with [Claude Code](https://claude.com/claude-code) — `/feature`
     EOF
     )"
     ```

3. Afficher l'URL de la PR.

---

## Phase 6 : Résumé

Présenter :
- **PR** : URL
- **Fichiers modifiés** : liste
- **Tests ajoutés** : nombre et couverture
- **Scope couvert** : rappel des objectifs cochés

Demander à l'utilisateur s'il veut :
- Lancer `/polish` sur la PR
- Merger directement
- Ajuster quelque chose

---

## Notes

- Ne JAMAIS modifier `CLAUDE.md`, `MEMORY.md`, ou les fichiers `.claude/`
- Si un objectif du scope s'avère trop complexe, le signaler et proposer de le sortir dans une PR séparée
- Respecter les contraintes projet-spécifiques décrites dans le CLAUDE.md du projet
