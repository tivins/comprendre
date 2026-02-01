# Introduction aux tests des requêtes MySQL complexes

## Pourquoi c’est particulier

Le code qui exécute des **requêtes MySQL complexes** (JOINs multiples, agrégations, GROUP BY, sous-requêtes) et qui **traite** ensuite les résultats en application (mapping, calculs, filtrage) cumule deux sources de régression : la requête elle-même (exactitude SQL, ordre, types) et la logique applicative (transformation des lignes, cas limites). Tester uniquement la couche applicative avec des mocks masque les erreurs SQL ; tester uniquement contre une base réelle sans maîtriser les données rend les tests fragiles ou non reproductibles.

En résumé : **il faut décider quoi tester (la requête, le traitement, les deux) et avec quel niveau d’isolation (mock, BDD de test, conteneur)** — et s’assurer que les données de test sont maîtrisées.

## Ce qu’on teste vraiment

- **La requête SQL** : qu’elle retourne les bonnes lignes, le bon ordre, les bons types et agrégats pour un jeu de données donné. Les erreurs typiques : JOIN incorrect, condition WHERE oubliée, alias ambigu, mauvaise agrégation.
- **Le traitement des données** : que l’application transforme correctement les lignes (mapping DTO, calculs, gestion du vide ou du nul). Souvent testable en unitaire en injectant un jeu de lignes fixe (mock du repository ou du résultat).
- **L’assemblage** : que l’exécution réelle (driver, connexion, transaction) et le traitement cohabitent correctement. Couvert par des tests d’intégration avec une BDD de test.

Chaque niveau répond à une question différente : « cette requête renvoie-t-elle le bon résultat pour ces données ? », « ce traitement transforme-t-il bien ces lignes ? », « le flux complet (requête + traitement) fonctionne-t-il avec une vraie MySQL ? ».

## Vocabulaire utile

- **Fixture** : jeu de données initial (INSERTs ou scripts) chargé avant les tests pour garantir un état reproductible.
- **Repository (ou Data Access)** : couche qui exécute la requête et retourne les lignes ; en l’injectant, on peut mocker le résultat en unitaire.
- **Test d’intégration BDD** : test qui lance une vraie base (locale, conteneur type Testcontainers) et vérifie requête + mapping + éventuellement transaction.
- **Snapshot testing** : comparaison du résultat (ex. JSON des lignes) à une version enregistrée. Pratique pour détecter les changements ; à utiliser avec discernement car les changements légitimes obligent à mettre à jour le snapshot.
- **Test flaky** : test qui passe ou échoue de façon non déterministe (données partagées, ordre d’exécution, timing). À éviter en particulier quand les données ne sont pas isolées par test.

Ces notions sont reprises et illustrées dans les chapitres suivants.

## Ce que cette doc ne couvre pas

- **Les tests de charge ou de performance** MySQL (benchmarks, EXPLAIN) : sujet distinct, même si une base de test peut servir aux deux.
- **L’optimisation SQL** en tant que telle : on se concentre sur la justesse des résultats et du traitement, pas sur le temps d’exécution.
- **Les procédures stockées** en détail : les principes (fixtures, intégration, isolation) s’appliquent, mais l’outillage peut différer selon le langage et le driver.

## Quand cette distinction compte

- **Stratégie de test** : décider combien de tests unitaires (traitement avec données mockées) vs intégration (requête + BDD), et où investir en priorité.
- **Diagnostic** : un échec en intégration mais pas en unitaire pointe vers la requête ou le schéma ; un échec uniquement en unitaire (avec données fixées) vers la logique de traitement.
- **CI et reproductibilité** : les tests d’intégration avec MySQL demandent une base prévisible (conteneur, fixtures) pour éviter les échecs aléatoires.

## Prochaines étapes

La suite de la documentation détaille :

- **[Concepts fondamentaux](./02-concepts-fondamentaux.md)** : découplage requête / traitement, types de tests applicables, reproductibilité
- **[Stratégies et outils](./03-strategies-et-outils.md)** : Testcontainers, mocks, assertions sur les jeux de données
- **[Exemples concrets](./04-exemples-concrets.md)** : rapports, agrégations, multi-tables, traitement après fetch
- **[Mise en pratique](./05-mise-en-pratique.md)** : quand quoi tester, pièges, intégration dans le workflow

---

**Rappel** : Bien distinguer « tester la requête », « tester le traitement » et « tester l’ensemble » permet de choisir le bon niveau de test et de garder des tests rapides et stables.
