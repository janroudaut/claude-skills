---
description: Génère ou met à jour le changelog métier (CHANGES_fr.md) à partir des commits récents
argument-hint: "full" pour tout régénérer (optionnel — par défaut, ajoute un nouveau jalon)
---

# /changelog — Mise à jour du changelog métier

Génère les entrées du changelog orienté métier (`CHANGES_fr.md`) à partir de l'historique Git.

**Arguments**: $ARGUMENTS

---

## Mode : Déterminer le mode d'exécution

- Si `$ARGUMENTS` contient `full` → **Mode complet** : régénérer tout le fichier
- Sinon → **Mode incrémental** : ajouter un nouveau jalon

---

## Mode incrémental (par défaut)

### Étape 1 : Lire l'état actuel

1. Lire `CHANGES_fr.md` pour identifier le dernier jalon (ex: `v3`)
2. Identifier la date du dernier jalon en parcourant les commits correspondants

### Étape 2 : Récupérer les commits récents

```bash
git log --oneline --no-merges main
```

Parcourir les commits depuis le dernier jalon connu. Si aucun commit significatif n'est trouvé, informer l'utilisateur et s'arrêter.

### Étape 3 : Analyser et regrouper

Pour chaque commit pertinent, lire le diff pour comprendre le changement :

```bash
git show --stat <sha>
git show <sha> -- <fichiers_clés>
```

Ignorer :
- Les commits de merge
- Les mises à jour de dépendances (dependabot)
- Les changements purement techniques (CI, config, refactoring interne) sauf s'ils sont significatifs
- Les modifications de tests uniquement

Regrouper les changements en catégories métier (Nouveautés, Qualité, Infrastructure).

### Étape 4 : Proposer le nouveau jalon

Déterminer le numéro de version (dernier + 1) et proposer un titre de jalon.

Générer le bloc au format :

```markdown
## vX — Titre du jalon (période)

Description courte du jalon (1-2 phrases).

**Nouveautés :**
- Item orienté utilisateur, pas technique
- Chaque item décrit le bénéfice, pas l'implémentation
```

### Étape 5 : Validation utilisateur

Présenter le bloc généré à l'utilisateur avec `AskUserQuestion` :
- Option 1 : "Valider et écrire" — insérer en haut du fichier (après le titre `# Historique des évolutions`)
- Option 2 : "Modifier le titre du jalon" — l'utilisateur propose un autre titre
- Option 3 : "Annuler"

### Étape 6 : Écrire

Si validé, insérer le nouveau jalon en haut de `CHANGES_fr.md`, juste après la ligne `# Historique des évolutions`, suivi d'une ligne vide et du séparateur `---`.

---

## Mode complet (`full`)

### Étape 1 : Récupérer tout l'historique

```bash
git log --oneline --no-merges --reverse main
```

### Étape 2 : Identifier les jalons

Regrouper les commits en jalons métier cohérents. Critères de découpage :
- Changement majeur de domaine fonctionnel
- Pause significative entre les périodes de développement
- Cohérence thématique des commits

### Étape 3 : Générer le fichier complet

Pour chaque jalon, générer un bloc au format standard. Numéroter séquentiellement (v1, v2, v3...).

### Étape 4 : Validation utilisateur

Présenter le fichier complet avec `AskUserQuestion` :
- Option 1 : "Valider et écrire"
- Option 2 : "Annuler"

### Étape 5 : Écrire

Si validé, réécrire `CHANGES_fr.md` entièrement.

---

## Règles de rédaction

- **Orienté métier** : décrire ce que l'utilisateur peut faire, pas comment c'est implémenté
- **Français** : tout en français, termes techniques courants acceptés (OCR, PDF, UUID)
- **Concis** : chaque item en une phrase, pas de jargon technique inutile
- **Pas de noms de fichiers/classes** : dire "analyse automatique des documents" pas "OcrLlmService"
- **Pas de numéros de PR/commit** dans le changelog final

---

## Notes

- Ne jamais écrire sans validation de l'utilisateur
- Si le nombre de commits est très important (> 100), demander à l'utilisateur de préciser une plage de dates ou un nombre de commits
- Le fichier utilise le titre `# Historique des évolutions` (pas "Journal des changements")
