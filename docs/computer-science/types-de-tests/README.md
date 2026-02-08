# Types de tests

## Introduction

Les **tests automatisés** couvrent différentes échelles : du comportement d’une fonction isolée à celui du système complet. Comprendre les types de tests (unitaire, intégration, bout en bout, etc.) permet de choisir quoi tester, où et avec quel outil — et d’éviter la confusion entre « tout tester » et « bien tester ».

Cette documentation s’adresse à des développeurs à l’aise avec le code et les environnements d’exécution, qui souhaitent clarifier les notions et les pratiques autour des tests.

## Objectifs de cette documentation

Cette documentation vous permettra de :

- Distinguer les principaux types de tests (unitaire, intégration, E2E, contractuels, etc.)
- Comprendre la pyramide des tests et son intérêt pratique
- Savoir quand privilégier tel type de test et quels compromis accepter
- Voir des exemples concrets en pseudo-code (Java/TypeScript)
- Mettre en pratique une stratégie de tests cohérente dans un projet

## Structure de la documentation

1. **[Introduction aux types de tests](./01-introduction.md)** — Pourquoi tester, ce que chaque niveau apporte, vocabulaire de base
2. **[Types et pyramide](./02-types-et-pyramide.md)** — Unitaire, intégration, E2E, contractuels ; pyramide et alternatives
3. **[Exemples concrets](./03-exemples-concrets.md)** — Cas métier et technique : service, API, base de données, front
4. **[Mise en pratique](./04-mise-en-pratique.md)** — Quand écrire quoi, pièges à éviter, intégration dans le workflow

---

**Note** : Les types de tests ne sont pas une fin en soi ; ils servent la confiance dans les changements et la maintenabilité. Le bon dosage dépend du contexte projet.

---

Liens :

* [Martin Fowler — Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html)
* [Google Testing Blog — Testing on the Toilet](https://testing.googleblog.com/)
