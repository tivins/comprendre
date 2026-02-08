# Concepts fondamentaux

Ce chapitre identifie les **couches testables** d'un moteur de recherche complexe, les **niveaux de test** adaptés à chacune, et les pistes de **découplage** réalistes dans un contexte Drupal 7 legacy.

## 1. Les couches d'un moteur de recherche

Un moteur de recherche custom, même monolithique, contient plusieurs responsabilités distinctes. Les identifier permet de choisir le bon niveau de test pour chacune.

### Construction de la requête (Query Building)

Le moteur traduit les paramètres utilisateur (filtres, tri, pagination) en requête SQL. C'est souvent la partie la plus complexe : conditions dynamiques, jointures conditionnelles, alias, sous-requêtes. Dans Drupal 7, cette construction passe par `db_select()` ou du SQL brut assemblé en PHP.

**Ce qu'on veut vérifier** : que pour un jeu de paramètres donné, la requête construite contient les bonnes clauses (WHERE, JOIN, ORDER BY). On ne vérifie pas ici que le résultat en base est correct — seulement que la « recette » de la requête est bonne.

### Gestion des tables temporaires

Le moteur crée des tables temporaires (résultats intermédiaires, scores, listes d'IDs pré-filtrés) pour découper la recherche en étapes. La logique de création, remplissage et utilisation de ces tables est une responsabilité distincte de la construction de la requête finale.

**Ce qu'on veut vérifier** : que les tables temporaires sont correctement créées et alimentées pour un état de base donné. Ce niveau est difficile à tester en unitaire (dépendance forte à la BDD) ; il relève typiquement du test d'intégration.

### Logique de filtrage et combinaison

Les filtres (taxonomie, date, champ custom, texte libre) sont combinables. Certains ont des dépendances (le filtre B n'est actif que si le filtre A est défini), des surcharges (un filtre peut en remplacer un autre selon le contexte), ou des valeurs par défaut. Cette logique de « quels filtres sont actifs et comment ils s'agencent » est distincte de la requête SQL elle-même.

**Ce qu'on veut vérifier** : que pour un ensemble de paramètres, les bons filtres sont sélectionnés, combinés et appliqués. Si cette logique est extraite dans une classe ou une fonction, elle est testable en unitaire pur — sans BDD, sans Drupal.

### Traitement des résultats

Les lignes SQL brutes (IDs de nœuds, scores, champs) passent par un traitement : chargement des entités Drupal, enrichissement, mise en forme, pagination. Ce traitement peut contenir des règles métier (exclusion de certains résultats, calcul de scores pondérés, regroupement).

**Ce qu'on veut vérifier** : que le traitement transforme correctement un jeu de résultats bruts en résultat final. Testable en unitaire si on peut injecter les données brutes.

---

## 2. Niveaux de test adaptés

### Test unitaire : la logique sans infrastructure

**Périmètre** : code PHP qui ne touche ni la BDD ni les APIs Drupal. Construction de filtres, combinaison de paramètres, traitement/transformation des résultats, calcul de scores.

**Prérequis** : que la logique soit **extraite** dans des classes ou fonctions qui reçoivent leurs données en paramètre (au lieu d'aller les chercher directement en base ou via des fonctions globales Drupal). C'est le travail de découplage — parfois minime, parfois conséquent.

**Apport** : feedback rapide, diagnostic précis (si le test échoue, c'est la logique PHP), pas de dépendance à l'environnement. C'est le niveau le moins cher en maintenance.

### Test d'intégration : requête + base réelle

**Périmètre** : exécution de la requête (ou de la chaîne construction → tables temporaires → requête finale) contre une MySQL avec un jeu de fixtures maîtrisé. On vérifie que la requête retourne les bons résultats (IDs, ordre, nombre).

**Prérequis** : une base de test avec des données reproductibles (fixtures). Peut nécessiter un bootstrap partiel de Drupal ou un accès direct à la base via PDO.

**Apport** : valide que le SQL est correct pour un état de base donné, détecte les régressions sur les jointures, les conditions, les tables temporaires. Plus lent qu'un unitaire, mais irremplaçable pour vérifier le SQL.

### Test fonctionnel : le pipeline complet

**Périmètre** : appel au moteur tel qu'un utilisateur le ferait (ou tel que le module l'appelle), de l'entrée des paramètres jusqu'au résultat final (nœuds chargés, paginés, mis en forme).

**Prérequis** : bootstrap Drupal complet, données en base (nœuds, taxonomie, champs), éventuellement un contexte HTTP simulé.

**Apport** : confiance maximale que « le truc marche de bout en bout ». Coût maximal aussi (lenteur, maintenance des fixtures complètes, fragilité face aux changements de schéma).

---

## 3. Découplage dans un contexte legacy

### Le problème du couplage

Dans un Drupal 7 legacy, la logique est souvent mêlée dans de longues fonctions procédurales : construction de requête, appels à `db_select()`, traitement des résultats, chargement de nœuds via `node_load_multiple()` — tout dans le même bloc. Tester en unitaire est alors impossible : la fonction ne peut pas tourner sans Drupal et sans base.

### Stratégie : extraire sans tout réécrire

L'idée n'est pas de refactoriser l'ensemble du module, mais d'**identifier les parties de logique pure** et de les sortir dans des fonctions ou classes testables :

**Extraire la logique de filtrage** : une fonction (ou classe) qui reçoit les paramètres bruts (tableau de filtres) et retourne une structure décrivant les filtres actifs, leurs valeurs, leurs dépendances. Pas de `db_select` dedans — que de la logique PHP.

```php
// Avant : tout mélangé dans une grosse fonction
function mymodule_search_execute($params) {
    $query = db_select('node', 'n');
    if (!empty($params['category'])) {
        $query->join('taxonomy_index', 'ti', 'n.nid = ti.nid');
        $query->condition('ti.tid', $params['category']);
    }
    // ... 200 lignes de plus
}

// Après : la logique de filtrage est extraite
class SearchFilterResolver {
    public function resolve(array $params): SearchFilters {
        $filters = new SearchFilters();
        if (!empty($params['category'])) {
            $filters->addTaxonomyFilter('category', $params['category']);
        }
        // ... logique de combinaison, dépendances, défauts
        return $filters;
    }
}
```

La classe `SearchFilterResolver` est testable en unitaire : on lui passe un tableau, on vérifie l'objet `SearchFilters` retourné. Pas de BDD, pas de Drupal.

**Extraire le traitement des résultats** : une fonction qui reçoit un tableau de lignes brutes (ou d'IDs) et retourne les résultats transformés (scores pondérés, exclusions, tri). Même logique : données en entrée, résultat en sortie, pas d'effet de bord.

```php
class SearchResultProcessor {
    public function process(array $rawRows, SearchFilters $filters): array {
        // Pondération, exclusion, regroupement...
        return $processedResults;
    }
}
```

**Laisser la construction SQL dans une couche dédiée** : un builder (ou la fonction existante, simplifiée) qui reçoit un objet `SearchFilters` et produit la requête. Cette couche reste couplée à la Database API Drupal — elle sera testée en intégration.

```php
class SearchQueryBuilder {
    public function build(SearchFilters $filters): SelectQuery {
        $query = db_select('node', 'n');
        foreach ($filters->getTaxonomyFilters() as $filter) {
            $query->join('taxonomy_index', 'ti_' . $filter->getField(), ...);
            $query->condition('ti_' . $filter->getField() . '.tid', $filter->getValue());
        }
        // ...
        return $query;
    }
}
```

### Compromis réaliste

On ne va pas tout extraire d'un coup. La priorité :

1. **Logique de filtrage/combinaison** : souvent la source principale de bugs (« tel filtre ne marche plus quand on l'active avec tel autre »). Haut retour sur investissement en tests unitaires.
2. **Traitement des résultats** : si des règles métier s'y trouvent (pondération, exclusion), les extraire et les tester en unitaire.
3. **Construction SQL** : plus couplée, testée en intégration. Le découplage consiste surtout à séparer la construction de l'exécution (le builder retourne un objet `SelectQuery`, pas un résultat).

Les parties qui restent couplées (hooks, tables temporaires, chargement de nœuds) continuent d'être testées en intégration — mais sur un périmètre mieux cerné.

---

## 4. Tables temporaires : un cas particulier

Les tables temporaires posent un défi spécifique :

- Elles n'existent que pendant la session MySQL (ou pendant la requête, selon le moteur).
- Leur contenu dépend de requêtes précédentes (remplissage depuis d'autres tables).
- Elles ne sont pas visibles après le test si la connexion est fermée.

**Stratégie de test** : la logique qui **décide quoi mettre** dans la table temporaire peut être extraite et testée en unitaire (c'est un builder de requête INSERT). L'exécution réelle (la table temporaire est créée, remplie, puis consommée) est un test d'intégration. On peut vérifier le contenu de la table temporaire pendant le test (SELECT sur la table avant qu'elle ne soit détruite) si le test s'exécute dans la même connexion.

Si les tables temporaires sont une source fréquente de régressions, c'est un argument pour les tester spécifiquement en intégration, avec des fixtures dédiées et des assertions sur le contenu intermédiaire.

---

## Synthèse

| Couche                      | Niveau de test recommandé | Prérequis de découplage             |
|-----------------------------|---------------------------|--------------------------------------|
| Logique de filtrage         | Unitaire                  | Extraire dans une classe/fonction   |
| Traitement des résultats    | Unitaire                  | Extraire dans une classe/fonction   |
| Construction de la requête  | Intégration               | Séparer construction et exécution   |
| Tables temporaires          | Intégration               | Tester dans la même connexion       |
| Pipeline complet            | Fonctionnel               | Bootstrap Drupal + fixtures         |

---

## Prochaines étapes

- **[Stratégies et approches](./03-strategies-et-approches.md)** : les méthodes concrètes (PHPUnit, fixtures, mocks, Testcontainers) pour chaque niveau
- **[Exemples concrets](./04-exemples-concrets.md)** : code PHP/Drupal illustrant chaque approche
- **[Mise en pratique](./05-mise-en-pratique.md)** : évaluer et améliorer l'existant

---

**Ressource** : Michael Feathers — [Working Effectively with Legacy Code](https://www.oreilly.com/library/view/working-effectively-with/0131177052/) — la référence sur le découplage progressif dans du code legacy.
