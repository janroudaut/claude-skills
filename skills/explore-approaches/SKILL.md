---
description: Explore plusieurs approches d'implémentation en parallèle via des agents concurrents, puis compare
argument-hint: description de la fonctionnalité ou du problème à résoudre
---

# /explore-approaches — Exploration Parallèle

Explore 2 approches d'implémentation concurrentes via des sub-agents, compare les résultats, et recommande la meilleure.

**Objectif**: $ARGUMENTS

## Principes

- **Concret** : chaque agent produit du CODE fonctionnel, pas un plan
- **Timeboxé** : max 15 tours par agent — pas de spirale d'exploration
- **Comparable** : mêmes critères d'évaluation pour chaque approche
- **Décision finale par l'utilisateur** : recommander, ne pas imposer

---

## Phase 1 : Cadrage (2 minutes max)

1. Lire les fichiers pertinents pour comprendre l'architecture existante
2. Identifier exactement 2 approches distinctes et viables
3. Présenter les 2 approches en 2-3 phrases chacune à l'utilisateur :

```
Approche A : [nom court] — [description]
Approche B : [nom court] — [description]
```

Utiliser `AskUserQuestion` pour confirmer ou ajuster les approches avant de lancer les agents.

---

## Phase 2 : Agents parallèles

Lancer 2 agents Task en parallèle avec `isolation: "worktree"` :

Chaque agent reçoit un prompt structuré :
```
Contexte : [description du projet et des fichiers pertinents]
Objectif : [la fonctionnalité demandée]
Approche : [A ou B — description détaillée]
Contraintes :
- Implémenter l'approche en modifiant les fichiers nécessaires
- Écrire au moins un test qui vérifie le comportement
- Lancer les tests modifiés pour vérifier qu'ils passent
- Résumer : fichiers modifiés, lignes de code ajoutées, complexité perçue
```

**Important** : utiliser `max_turns: 15` pour timeboxer chaque agent.

---

## Phase 3 : Comparaison

Quand les 2 agents terminent, comparer sur ces critères :

| Critère | Approche A | Approche B |
|---------|-----------|-----------|
| Fichiers modifiés | N | N |
| Lignes ajoutées | N | N |
| Tests écrits | N | N |
| Tests passent ? | oui/non | oui/non |
| Cohérence avec l'existant | score /5 | score /5 |
| Complexité | faible/moyenne/élevée | faible/moyenne/élevée |

---

## Phase 4 : Recommandation

Présenter le tableau de comparaison et recommander l'approche qui :
1. A le moins de fichiers modifiés
2. Est la plus cohérente avec les patterns existants du projet
3. A les tests qui passent

Utiliser `AskUserQuestion` :
```
label: "Approche A — [nom]"
description: "[résumé des avantages/inconvénients]"
```

---

## Phase 5 : Application

Après le choix de l'utilisateur :
1. Appliquer les changements du worktree choisi dans la branche courante
2. Relancer les tests pour confirmer
3. Résumer les fichiers modifiés

Ne PAS commiter automatiquement — attendre que l'utilisateur le demande.

---

## Notes

- Si les 2 approches échouent (tests cassés), le dire franchement et proposer une 3e piste
- Ne pas favoriser une approche pour des raisons esthétiques — prioriser ce qui fonctionne
- Les worktrees sont automatiquement nettoyés si non utilisés
