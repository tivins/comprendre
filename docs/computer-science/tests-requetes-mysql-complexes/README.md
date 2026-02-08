# Tests des requêtes MySQL complexes et du traitement des données

## Introduction

Tester du code qui s’appuie sur des **requêtes MySQL complexes** (JOINs, agrégations, sous-requêtes) et sur le **traitement applicatif** des résultats pose des défis spécifiques : reproductibilité des données, coût des tests d’intégration, découplage entre la logique SQL et la logique métier. Les méthodes courantes et recommandées permettent de choisir le bon niveau de test (unitaire avec mocks, intégration avec une vraie base) et d’éviter les pièges (tests flaky, données fantômes, sur-dépendance au schéma).

Cette documentation s’adresse à des développeurs à l’aise avec SQL et un langage applicatif (Java, PHP, TypeScript, etc.), qui souhaitent structurer les tests autour des requêtes complexes et du traitement des données.

## Objectifs de cette documentation

Cette documentation vous permettra de :

- Comprendre pourquoi les requêtes MySQL complexes et le traitement des données demandent une stratégie de test adaptée
- Distinguer les approches : découplage requête / traitement, tests unitaires avec mocks, tests d’intégration avec BDD réelle ou conteneur
- Connaître les méthodes recommandées : fixtures, isolation des données, assertions sur les jeux de résultats
- Voir des exemples concrets (rapports, agrégations, multi-tables, traitement métier après fetch)
- Mettre en pratique une stratégie cohérente (CI, reproductibilité, pièges à éviter)

## Structure de la documentation

1. **[Introduction](./01-introduction.md)** — Enjeux des tests autour des requêtes complexes, ce qu’on teste vraiment, vocabulaire utile
2. **[Concepts fondamentaux](./02-concepts-fondamentaux.md)** — Découplage requête / traitement, types de tests applicables, reproductibilité des données
3. **[Stratégies et outils](./03-strategies-et-outils.md)** — Testcontainers, BDD de test, mocks des résultats, assertions sur les données
4. **[Exemples concrets](./04-exemples-concrets.md)** — Rapport avec agrégation, requête multi-tables, traitement métier après fetch
5. **[Mise en pratique](./05-mise-en-pratique.md)** — Quand privilégier quelle approche, pièges courants, intégration dans le workflow et la CI

---

**Note** : Tester les requêtes complexes et le traitement des données ne signifie pas tout faire en intégration ; le découplage et le choix du niveau de test (unitaire vs intégration) restent essentiels pour garder des tests rapides et maintenables.

---

Liens :

* [Martin Fowler — Practical Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html)
* [Testcontainers — MySQL Module](https://testcontainers.com/modules/databases/mysql/)
* [MySQL — Testing](https://dev.mysql.com/doc/)
