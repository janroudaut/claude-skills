---
description: Diagnostic et correction de bug en mode test-first — reproduire, corriger, vérifier, itérer
argument-hint: description du bug (symptôme observé)
---

# /bugfix — Correction Test-First

Diagnostique et corrige un bug en suivant un cycle strict : reproduire avec un test, corriger, vérifier.

**Bug**: $ARGUMENTS

**Pré-requis** : consulter le CLAUDE.md du projet pour les commandes de test et les conventions du framework.

## Principes

- **Test-first** : JAMAIS de correction sans test de reproduction qui échoue d'abord
- **Root cause** : trouver la vraie cause, pas un contournement
- **Itératif** : si le fix ne marche pas, changer de stratégie (pas de variations mineures)
- **Autonome total** : itérer jusqu'au vert SANS demander à l'utilisateur. Ne s'arrêter que si toutes les stratégies sont épuisées

---

## Phase 1 : Diagnostic et test (agents parallèles)

Lancer 2 agents Task en parallèle :

### Agent A — Diagnostic
1. Lire les fichiers pertinents mentionnés dans la description du bug
2. Tracer le flux de données complet (save → load → render)
3. Vérifier les DEUX côtés (client ET serveur)
4. Retourner : cause probable, fichier:ligne suspecte, stratégie de fix recommandée

### Agent B — Test de reproduction
1. Identifier le bon fichier de test (system test si UI, unit/integration sinon)
2. Écrire un test minimal qui **reproduit exactement le symptôme**
3. Lancer le test pour confirmer qu'il échoue (utiliser la commande de test du projet)

4. Vérifier que le test échoue **pour la bonne raison** (pas une erreur de setup)
5. Retourner : le test et sa sortie d'erreur

Si l'agent B n'arrive pas à reproduire le bug :
- Utiliser les findings de l'agent A pour ajuster l'angle
- Essayer un 2e test différent
- Si impossible à reproduire après 2 tentatives, signaler à l'utilisateur

**Output** : Diagnostic + test qui échoue.

---

## Phase 2 : Boucle de correction autonome

**Ne PAS demander à l'utilisateur — itérer jusqu'au vert.**

1. Implémenter la correction **minimale** — ne toucher que le code nécessaire
2. Relancer le test de reproduction

### Si le test échoue encore :

- **Ne PAS** réessayer la même approche avec des variations mineures
- Analyser POURQUOI le fix n'a pas marché
- Choisir une stratégie **fondamentalement différente**
- Réessayer — boucler jusqu'à 5 stratégies différentes

### Si le test passe :

Passer à la Phase 3.

### Si 5 stratégies échouent :

S'arrêter et présenter à l'utilisateur :
- Les 5 approches tentées et pourquoi chacune a échoué
- L'hypothèse révisée sur la cause du bug
- Une suggestion pour débloquer

---

## Phase 3 : Vérification complète

1. Lancer la suite de tests complète pour détecter les régressions
2. Si des tests existants cassent :
   - Analyser si c'est une régression réelle ou un test fragile
   - Corriger les régressions réelles
   - Signaler les tests fragiles sans les modifier

---

## Phase 4 : Résumé

Présenter :
- **Cause** : la root cause en 1-2 phrases
- **Fix** : ce qui a été modifié (fichiers et lignes)
- **Test** : le test ajouté
- **Régressions** : aucune / liste des problèmes détectés

Ne PAS commiter automatiquement — attendre que l'utilisateur le demande.

---

## Notes

- Respecter les conventions de test du projet (voir CLAUDE.md pour les helpers et contraintes spécifiques)
