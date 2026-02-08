# Introduction

## Le problème

Un moteur de recherche custom dans Drupal 7, c'est un mélange de plusieurs choses difficiles à tester :

- **Requêtes SQL complexes** construites dynamiquement : jointures multiples, sous-requêtes, agrégations, parfois du SQL brut combiné avec la Database API Drupal (`db_select`, `db_query`).
- **Tables temporaires** créées à la volée pour stocker des résultats intermédiaires, recoupés ensuite par d'autres requêtes.
- **Filtres dynamiques** : l'utilisateur combine des critères (type de contenu, taxonomie, champs custom, plage de dates, texte libre…). Chaque combinaison produit une requête différente.
- **Surcharges et hooks** : d'autres modules ou le thème peuvent altérer la requête (`hook_query_alter`, callbacks), ajoutant des conditions ou des jointures.
- **Couplage à Drupal** : les fonctions de la Database API, les appels à `node_load`, les tables `node`, `field_data_*`, `taxonomy_index` sont omniprésents.

Le résultat : un code qui fait « beaucoup de choses » — construire la requête, l'exécuter, traiter les résultats — souvent dans les mêmes fonctions, avec un couplage fort à l'infrastructure.

## Ce qu'il faut tester (et pourquoi c'est compliqué)

Trois familles de comportements doivent être couvertes :

### 1. La construction des requêtes

Quand l'utilisateur demande « tous les articles de la catégorie X publiés après le 1er janvier, triés par pertinence », le moteur doit produire la bonne requête (les bonnes jointures, les bonnes conditions WHERE, le bon ORDER BY). Les régressions typiques : une jointure oubliée quand un filtre est désactivé, une condition manquante pour un cas particulier, un alias en conflit.

### 2. L'exécution et les résultats

La requête, une fois exécutée sur un jeu de données, doit retourner les bons résultats : les bons nœuds, dans le bon ordre, avec les bonnes valeurs. Cela inclut la logique des tables temporaires (est-ce qu'elles sont correctement remplies et utilisées ?) et le comportement aux limites (aucun résultat, filtre vide, combinaisons rares).

### 3. Le traitement des résultats

Les résultats bruts (IDs, lignes SQL) passent par un traitement : chargement des nœuds, formatage, pagination, éventuellement enrichissement (champs calculés, scores). Les régressions typiques : mauvais mapping, perte de données lors du formatage, pagination incorrecte.

## L'approche actuelle : PHPUnit + données pré-insérées

L'équipe dispose de tests PHPUnit qui :

1. Insèrent des données en base (nœuds, termes de taxonomie, champs) avant le test.
2. Appellent le moteur de recherche avec des paramètres de filtre.
3. Vérifient que les résultats (IDs des nœuds, nombre, ordre) correspondent aux attentes.

C'est une approche qui **fonctionne** — elle détecte les régressions et donne de la confiance — mais elle présente des limites qu'il faut connaître.

### Ce qui marche

- **Réalisme** : on teste contre une vraie base, avec de vraies données Drupal. Les requêtes sont exécutées telles qu'elles le seront en production.
- **Couverture fonctionnelle** : si les jeux de données sont bien conçus, on couvre les cas importants (combinaisons de filtres, cas vides, données liées).
- **Filet de sécurité** : même imparfaits, ces tests signalent les régressions majeures.

### Ce qui pose problème

- **Lenteur** : chaque test insère des données, exécute des requêtes, parfois bootstrap Drupal. Sur un moteur complexe avec des dizaines de combinaisons de filtres, la suite devient lente.
- **Fragilité** : une modification du schéma (ajout d'un champ, changement de table) peut casser beaucoup de tests d'un coup, même si le comportement métier n'a pas changé.
- **Diagnostic difficile** : quand un test échoue, il est difficile de savoir si c'est la requête qui est incorrecte, les données de test qui sont incomplètes, ou le traitement des résultats qui a régressé. Tout est testé en bloc.
- **Maintenance lourde** : les fixtures (données insérées) doivent être maintenues en cohérence avec le schéma, les types de contenu, les vocabulaires de taxonomie. Tout changement dans la structure Drupal impose de revoir les données de test.
- **Couplage au framework** : les tests dépendent du bootstrap Drupal, de la Database API, des tables internes. Ils ne testent pas la logique métier de manière isolée.

En résumé : **ces tests sont des tests d'intégration (voire fonctionnels) déguisés en tests unitaires**. Ils n'utilisent pas PHPUnit « en mode unitaire » (isolation, mocks, test d'une seule responsabilité) mais comme un runner de tests d'intégration. Ce n'est pas forcément un problème — c'est un choix — mais il faut le savoir pour compléter avec d'autres niveaux de test.

## Ce que cette doc propose

Plutôt qu'une refonte, une **stratégie progressive** :

- **Découper** ce qui est testable en unitaire (construction de requêtes, logique de filtres, traitement des résultats) et ce qui nécessite une base (exécution réelle des requêtes, tables temporaires).
- **Renforcer** les tests d'intégration existants (fixtures minimales, isolation, assertions ciblées).
- **Ajouter** des tests unitaires là où le retour sur investissement est immédiat (logique de construction, règles métier sur les résultats).
- **Accepter** que certaines parties (interactions Drupal pures, hooks) restent testées en intégration — et structurer ces tests pour qu'ils soient maintenables.

## Prochaines étapes

- **[Concepts fondamentaux](./02-concepts-fondamentaux.md)** : les couches testables d'un moteur de recherche, niveaux de test, découplage dans un contexte legacy
- **[Stratégies et approches](./03-strategies-et-approches.md)** : tests unitaires, intégration, fonctionnel — quoi et comment
- **[Exemples concrets](./04-exemples-concrets.md)** : code PHP/Drupal illustrant chaque approche
- **[Mise en pratique](./05-mise-en-pratique.md)** : évaluer et améliorer l'existant

---

**Rappel** : L'objectif n'est pas d'atteindre 100 % de couverture, mais de **savoir ce qu'on teste, à quel niveau, et pourquoi** — pour que chaque test apporte de la confiance sans devenir un fardeau de maintenance.
