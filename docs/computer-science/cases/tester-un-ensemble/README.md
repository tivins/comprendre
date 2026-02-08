# tester les ensembles

## Introduction

Tester un **moteur de recherche custom** bâti sur Drupal 7 — requêtes dynamiques, tables temporaires, filtres combinables, jointures multiples — demande une stratégie adaptée. Les tests « classiques » (lancer le moteur contre une base pré-remplie et vérifier les résultats) ne suffisent pas : ils mélangent les niveaux, deviennent fragiles et ne localisent pas précisément les régressions.

Cette documentation part d'un cas concret (module Drupal 7, codebase partiellement legacy, principes SOLID en cours d'adoption) pour expliquer **comment structurer les tests** d'un tel moteur : quoi tester à chaque niveau (unitaire, intégration, fonctionnel), comment isoler les différentes couches, et comment améliorer progressivement une suite de tests existante.

## Objectifs de cette documentation

Cette documentation vous permettra de :

- Identifier les **couches testables** d'un moteur de recherche complexe (construction de requêtes, filtres, tables temporaires, traitement des résultats)
- Distinguer les **niveaux de test** adaptés : unitaire (logique PHP), intégration (requêtes + BDD), fonctionnel (pipeline complet)
- Comprendre les **spécificités Drupal 7** qui compliquent les tests (couplage aux APIs Drupal, `db_select`, hooks, tables temporaires)
- Évaluer une approche existante (PHPUnit + données pré-insérées) et la **faire évoluer**
- Mettre en place une stratégie de tests progressive et réaliste pour un codebase legacy

## Structure de la documentation

1. **[Introduction](./01-introduction.md)** — Contexte Drupal 7, nature du problème, diagnostic de l'approche actuelle
2. **[Concepts fondamentaux](./02-concepts-fondamentaux.md)** — Couches testables d'un moteur de recherche, niveaux de test, découplage dans un contexte legacy
3. **[Stratégies et approches](./03-strategies-et-approches.md)** — Tests unitaires (builders, filtres), intégration (BDD réelle, fixtures), fonctionnel (pipeline), gestion des tables temporaires
4. **[Exemples concrets](./04-exemples-concrets.md)** — Code PHP/Drupal : test d'un builder de filtre, test d'intégration avec fixtures, test du pipeline de recherche
5. **[Mise en pratique](./05-mise-en-pratique.md)** — Évaluation de l'approche actuelle, plan d'amélioration progressif, intégration CI, pièges à éviter

---

**Note** : Cette documentation ne prône pas une refonte complète du moteur ni un passage à Drupal 10. Elle propose une stratégie de tests progressive, applicable au code existant, en améliorant le découplage au fur et à mesure.

---

Liens :

* [PHPUnit — Documentation officielle](https://docs.phpunit.de/)
* [Drupal 7 — Database API](https://www.drupal.org/docs/7/api/database-api)
* [Martin Fowler — Practical Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html)
* [Michael Feathers — Working Effectively with Legacy Code](https://www.oreilly.com/library/view/working-effectively-with/0131177052/)
