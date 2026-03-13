---
description: Détecte le drift entre le code et la documentation, propose et applique les mises à jour nécessaires
argument-hint: chemin ou nom du fichier doc à mettre à jour (optionnel — sans argument, scanne le projet)
---

# /update-doc — Mise à jour de la documentation

Analyse les écarts entre le code actuel et la documentation, puis propose et applique les mises à jour nécessaires.

**Scope**: $ARGUMENTS

## Principes

- **Non-intrusif** : ne jamais modifier sans validation de l'utilisateur
- **Fidèle au style** : respecter la structure et le ton des docs existantes
- **Discovery-based** : pas de scopes hardcodés — scanner le projet pour trouver la doc
- **Orienté lecteur** : adapter le niveau de détail au public cible du document

---

## Phase 1 : Découverte et sélection du scope

### Si `$ARGUMENTS` est fourni :
- Si c'est un chemin de fichier → cibler ce fichier directement
- Si c'est un mot-clé → chercher les fichiers doc correspondants (`README`, `api`, `setup`, etc.)

### Si `$ARGUMENTS` est vide :

1. Scanner le projet pour trouver les fichiers de documentation :
   ```bash
   find . -name "*.md" -not -path "./.claude/*" -not -path "./vendor/*" -not -path "./node_modules/*" | head -30
   ```

2. Présenter les fichiers trouvés via `AskUserQuestion` avec `multiSelect: true` :
   - Regrouper par dossier si beaucoup de fichiers
   - Option "Tout scanner" pour une analyse complète

**Attendre la sélection avant de continuer.**

---

## Phase 2 : Analyse des écarts

Pour chaque fichier doc sélectionné :

### 2A. Lire le contenu existant

Lire le fichier Markdown en entier.

### 2B. Identifier le code lié

Inférer les fichiers source documentés en cherchant :
- Les chemins de fichiers mentionnés dans la doc
- Les classes, méthodes, commandes référencées
- Les noms de fichiers/dossiers cités

Explorer le code correspondant pour comprendre l'état actuel.

### 2C. Détecter les écarts

Pour chaque section de la doc, repérer :
- **Informations obsolètes** : commandes, chemins, options, noms qui ont changé dans le code
- **Fonctionnalités non documentées** : présentes dans le code mais absentes de la doc
- **Fonctionnalités supprimées** : documentées mais supprimées du code
- **Incohérences** : versions, dépendances, exemples qui ne correspondent plus

---

## Phase 3 : Proposition des changements

### 3A. Résumé des écarts

Présenter un résumé textuel clair de tous les écarts trouvés, organisé par fichier.

### 3B. Choix des modifications

Utiliser `AskUserQuestion` avec `multiSelect: true` pour lister les modifications proposées.

Format de chaque option :
```
label: "fichier.md — Description courte du changement"
description: "Détail : ce qui est obsolète / manquant et ce qui sera modifié"
```

Inclure les options :
- **"Tout appliquer"** : applique toutes les modifications proposées
- **"Annuler"** : ne rien modifier

Si plus de 4 modifications, présenter les 4 plus importantes via `AskUserQuestion` et lister les autres en texte. L'utilisateur peut répondre "Other" pour sélectionner parmi les restantes.

**Attendre la sélection de l'utilisateur avant de continuer.**

Si l'utilisateur choisit "Annuler", s'arrêter avec un message de confirmation.

---

## Phase 4 : Application des modifications

Pour chaque modification sélectionnée :
1. Lire le fichier cible
2. Appliquer la modification (Edit ou Write selon l'ampleur)
3. Respecter le style existant : titres, listes, formatage, niveau de détail, langue

---

## Phase 5 : Vérification

### 5A. Relecture

Relire chaque fichier modifié pour vérifier :
- Cohérence du contenu
- Pas de coquilles ou accents manquants (si doc en français)
- Formatage Markdown correct

### 5B. Vérification des liens

Vérifier que :
- Les liens vers des fichiers ou images pointent vers des ressources existantes
- Les ancres internes (`[...](#section)`) sont valides
- Les références croisées entre fichiers de doc sont correctes

### 5C. Résumé final

Présenter à l'utilisateur :
- La liste des fichiers modifiés
- Un résumé de chaque modification appliquée
- Les éventuels problèmes non résolus

Ne PAS commiter automatiquement — attendre que l'utilisateur le demande.

---

## Notes

- Si aucun écart n'est trouvé, le dire simplement — ne pas inventer de modifications
- Ne jamais écrire de détails d'implémentation dans des docs destinées aux utilisateurs finaux
- Respecter la langue du document existant (ne pas traduire un doc EN en FR ou inversement)
