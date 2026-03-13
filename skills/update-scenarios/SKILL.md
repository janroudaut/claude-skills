---
description: Intègre les nouvelles fonctionnalités dans les scénarios e2e existants pour maintenir des parcours utilisateur "idéaux" à jour
argument-hint: fonctionnalité ajoutée/modifiée (ex. "contrôle qualité à la réception")
---

# /update-scenarios — Mise à jour des Scénarios E2E

Quand une fonctionnalité est ajoutée ou modifiée, cette skill l'intègre dans les scénarios de démo e2e existants pour que les parcours utilisateur restent à jour et enregistrables avec `bin/demo --record`.

Chaque scénario correspond à un **domaine** ou **focus** du workflow de l'application (ex. réception, livraison, facturation). Ils peuvent être lancés ensemble ou isolément.

**Fonctionnalité**: $ARGUMENTS

**Pré-requis** : consulter le CLAUDE.md du projet pour les commandes de test, la structure du code (routes, vues, contrôleurs) et les conventions du framework.

## Principes

- **Intégration, pas isolation** : la nouvelle feature s'insère dans le workflow existant du bon domaine, pas dans un test orphelin
- **Parcours idéal** : chaque scénario montre le parcours typique d'un utilisateur dans un domaine donné
- **Multi-domaine** : une appli a plusieurs scénarios (un par domaine/focus), exécutables ensemble (`bin/demo all`) ou séparément (`bin/demo reception`)
- **Lisible en vidéo** : noms de tests humains et narratifs — ils apparaissent comme chapitres MKV et dans la barre en bas

---

## Phase 1 : Comprendre la feature et son domaine

### 1A. Analyser la fonctionnalité

1. Identifier les fichiers de la feature : routes, contrôleurs/handlers, vues/templates, JS
2. Tracer le parcours utilisateur : à quel moment du workflow cette feature intervient-elle ?
3. Identifier les interactions : formulaires, modals, sélections, clics, feedback visuel

### 1B. Cartographier les scénarios existants

1. Lire les scénarios de démo actuels :
   ```bash
   find test/ tests/ spec/ -name "demo_*" -o -name "*_workflow_*" -o -name "*_scenario_*" -o -name "*_e2e_*" 2>/dev/null
   ```
2. Lire `bin/demo` (si existant) pour comprendre les domaines couverts :
   - Quels targets existent ? (ex. `e2e`, `reception`, `delivery`)
   - Quels workflows chaque target couvre ?
3. Pour chaque scénario, lister les étapes du parcours

### 1C. Identifier le domaine cible

Déterminer **dans quel scénario** la feature s'insère :
- Feature liée à la réception → scénario `reception`
- Feature transversale (ex. notifications) → plusieurs scénarios impactés
- Feature d'un nouveau domaine → nouveau scénario à créer

Puis trouver le **point d'insertion** dans le parcours :

```
Workflow "réception" existant :  [Création] → [Réception physique] → [Validation] → [Clôture]
                                                                ↑
Nouvelle feature :                                         [Contrôle QC]
                                                                ↓
Workflow mis à jour :            [Création] → [Réception physique] → [Contrôle QC] → [Validation] → [Clôture]
```

---

## Phase 2 : Proposition

Présenter via `AskUserQuestion` :

- Quel(s) scénario(s) existant(s) sont impactés (domaine)
- Où la feature s'insère dans chaque parcours
- Si un nouveau chapitre (= test) est nécessaire ou si ça tient dans un test existant
- Si un nouveau domaine/scénario doit être créé

Options :
- "Valider l'intégration" (Recommandé)
- "Ajuster"

**Règle de découpage** : si l'ajout allonge un test de plus de ~20 secondes de vidéo, extraire un nouveau test (= nouveau chapitre MKV).

**Attendre la validation avant de modifier le code.**

---

## Phase 3 : Mise à jour des scénarios

### Modifier un test existant

Insérer les nouvelles étapes au bon endroit dans le parcours :

```ruby
test "Réception complète avec validation" do
  # ... étapes existantes de création ...
  demo_pause

  # ── NOUVEAU : Contrôle qualité ──
  click_on "Lancer le contrôle"
  demo_pause

  within_modal do
    select "Conforme", from: "Résultat"
    demo_tick
    click_on "Valider"
  end
  demo_pause

  assert_text "Contrôle validé"
  demo_pause
  # ── FIN NOUVEAU ──

  # ... étapes existantes de clôture ...
end
```

### Ajouter un nouveau chapitre (test) dans un scénario existant

Si la feature justifie son propre chapitre vidéo :

```ruby
test "Contrôle qualité — anomalie signalée et traitée" do
  reception = setup_reception_pending_qc

  sign_in_as("responsable_qc")
  demo_pause

  visit reception_path(reception)
  demo_pause

  # ... interactions QC ...
  demo_pause(4)  # montrer le résultat
end
```

### Créer un nouveau scénario (nouveau domaine)

Si la feature ouvre un nouveau domaine de l'application :

1. Créer le fichier de test e2e pour le domaine
2. Concevoir le parcours complet du domaine (pas juste la feature)
3. Ajouter dans `bin/demo` : `target_file`, `target_title`, `expand_targets`

### Noms de tests : humains et narratifs

Les noms apparaissent dans la vidéo. Écrire comme un titre de chapitre :

```ruby
# ✗ Technique
test "reception_with_qc_check_and_anomaly_report"

# ✓ Narratif — c'est un titre de chapitre vidéo
test "Contrôle qualité — anomalie signalée et traitée"
```

### Rythme des interactions

| Action | Pause | Helper |
|--------|-------|--------|
| Chargement de page, soumission | 2-3s | `demo_pause` |
| Clic, focus, saisie | 0.3-0.5s | `demo_tick` |
| Résultat important à voir | 3-5s | `demo_pause(4)` |

### Données de test

- Adapter le `setup` existant du scénario pour inclure les données de la feature
- Si les données sont complexes, ajouter un helper privé `setup_<context>_data`
- Noms réalistes : `"Transports Martin"` pas `"Test 1"` — l'UI doit sembler en production

---

## Phase 4 : Intégration avec bin/demo

### Si un nouveau scénario (domaine) est créé :

1. Ajouter dans `target_file()` : mapping target → fichier test
2. Ajouter dans `target_title()` : titre humain pour les title cards
3. Inclure dans `expand_targets()` (composite `all`)
4. Mettre à jour `--list`

### Si seuls des tests existants sont modifiés :

Rien à changer dans `bin/demo` — les chapitres MKV se mettent à jour automatiquement via les noms de tests.

### Si `bin/demo` n'existe pas :

Proposer d'installer le template depuis claude-skills.

---

## Phase 5 : Vérification

1. Lancer le scénario modifié (commande de test du projet)

2. Vérifier visuellement si possible :
   ```bash
   bin/demo <target>
   ```

3. Vérifier la cohérence du parcours : les étapes s'enchaînent-elles naturellement ?

---

## Phase 6 : Résumé

Présenter :
- **Domaine(s) impacté(s)** : quels scénarios modifiés
- **Point d'insertion** : où la feature s'intègre dans le workflow
- **Chapitres ajoutés/modifiés** : noms des tests (= titres vidéo)
- **Nouveau domaine** : si un scénario a été créé, sa target `bin/demo`

Ne PAS commiter automatiquement — attendre que l'utilisateur le demande.

---

## Notes

- L'objectif est de maintenir des **parcours idéaux par domaine** — pas une collection de tests isolés
- Chaque domaine = un scénario = une target `bin/demo` = une vidéo avec chapitres
- Si un helper d'interaction manque (drag-drop, right-click, canvas), le signaler plutôt que l'inventer
- Les scénarios de démo complètent les tests fonctionnels — ils ne les remplacent pas
