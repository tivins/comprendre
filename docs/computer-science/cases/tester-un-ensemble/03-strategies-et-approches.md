# Stratégies et approches

Ce chapitre détaille les **méthodes concrètes** pour tester un moteur de recherche Drupal 7 à chaque niveau : tests unitaires (logique PHP extraite), tests d'intégration (requêtes + BDD réelle), tests fonctionnels (pipeline complet). Avec un focus sur les spécificités Drupal 7.

## 1. Tests unitaires : la logique extraite

### Quoi tester en unitaire

Tout ce qui est **logique PHP pure** — sans appel à la BDD ni aux APIs Drupal :

- **Résolution des filtres** : pour un tableau de paramètres, quels filtres sont actifs ? quelles dépendances ? quelles valeurs par défaut ? Ce sont des règles métier : elles méritent des tests dédiés.
- **Combinaison et priorité des filtres** : si le filtre A est actif, le filtre B est ignoré ; si aucun filtre n'est défini, on applique les défauts. Ce genre de logique est une source fréquente de bugs.
- **Traitement des résultats** : pondération de scores, exclusion de nœuds, dédoublonnage, tri applicatif, pagination calculée. Si ces traitements sont extraits, on les teste avec un simple tableau PHP en entrée.
- **Mapping et transformation** : conversion d'un résultat SQL brut en DTO ou en structure métier.

### Comment tester en unitaire

PHPUnit classique, **sans bootstrap Drupal**. Le test instancie directement la classe sous test, lui passe des données en paramètre et vérifie le résultat.

```php
class SearchFilterResolverTest extends \PHPUnit\Framework\TestCase
{
    public function testCategoryFilterIsAppliedWhenPresent(): void
    {
        $resolver = new SearchFilterResolver();
        $filters = $resolver->resolve(['category' => 42, 'date_from' => '2024-01-01']);

        $this->assertTrue($filters->has('category'));
        $this->assertEquals(42, $filters->get('category')->getValue());
    }

    public function testEmptyParamsReturnDefaultFilters(): void
    {
        $resolver = new SearchFilterResolver();
        $filters = $resolver->resolve([]);

        $this->assertTrue($filters->has('status'));
        $this->assertEquals('published', $filters->get('status')->getValue());
    }
}
```

Pas de mock complexe, pas de base de données. Si le test échoue, le problème est dans la logique de filtrage — pas dans la requête ni dans Drupal.

### Quand mocker

Le mock est utile quand la classe sous test a une **dépendance injectée** qu'on ne veut pas activer en test :

```php
class SearchResultProcessorTest extends \PHPUnit\Framework\TestCase
{
    public function testExcludesNodesBelowScoreThreshold(): void
    {
        // Le scorer est mocké : on contrôle les scores retournés
        $scorer = $this->createMock(RelevanceScorer::class);
        $scorer->method('score')->willReturnMap([
            [101, 0.9],
            [102, 0.2],
            [103, 0.7],
        ]);

        $processor = new SearchResultProcessor($scorer, $minScore = 0.5);
        $result = $processor->process([
            ['nid' => 101], ['nid' => 102], ['nid' => 103],
        ]);

        $this->assertCount(2, $result);
        $this->assertEquals([101, 103], array_column($result, 'nid'));
    }
}
```

Attention : **ne pas mocker la classe sous test**, et ne pas sur-mocker des dépendances triviales (un simple DTO, un tableau). Le mock sert à isoler une dépendance dont le comportement est complexe ou lent.

---

## 2. Tests d'intégration : requêtes + BDD réelle

### Pourquoi une vraie MySQL

Les requêtes du moteur utilisent des fonctionnalités spécifiques MySQL (ou MariaDB) : `CREATE TEMPORARY TABLE`, `GROUP_CONCAT`, expressions conditionnelles, comportements d'alias, `COLLATE`. Tester contre SQLite ou H2 masquerait des incompatibilités. Pour un moteur de recherche complexe, la **base réelle** est indispensable au niveau intégration.

### Deux approches

#### A. Bootstrap Drupal + base de test (DrupalWebTestCase / custom)

Drupal 7 propose `DrupalWebTestCase` (Simpletest) qui crée un site Drupal de test dans une base préfixée. C'est lourd (bootstrap complet, création de tables, installation de modules) mais réaliste : les tables Drupal existent, les APIs fonctionnent.

Dans un contexte PHPUnit, on peut reproduire l'idée : un **bootstrap partiel** de Drupal dans le `setUp()` qui initialise la connexion et les tables nécessaires, puis insère les fixtures.

```php
abstract class SearchIntegrationTestCase extends \PHPUnit\Framework\TestCase
{
    protected static $connection;

    public static function setUpBeforeClass(): void
    {
        // Bootstrap Drupal (ou au minimum la Database API)
        define('DRUPAL_ROOT', '/path/to/drupal');
        require_once DRUPAL_ROOT . '/includes/bootstrap.inc';
        drupal_bootstrap(DRUPAL_BOOTSTRAP_DATABASE);

        static::$connection = \Database::getConnection();
    }

    protected function setUp(): void
    {
        // Démarrer une transaction pour rollback après chaque test
        static::$connection->query('START TRANSACTION');
    }

    protected function tearDown(): void
    {
        static::$connection->query('ROLLBACK');
    }
}
```

Le rollback en `tearDown` garantit l'isolation entre tests : chaque test part du même état de base.

#### B. Base MySQL dédiée (sans Drupal)

Si le moteur est suffisamment découplé (le builder produit du SQL, l'exécution se fait via PDO), on peut tester contre une MySQL locale ou conteneurisée sans charger Drupal. On recrée les tables nécessaires (`node`, `field_data_*`, `taxonomy_index`, les tables custom) et on insère les fixtures.

Avantage : **beaucoup plus rapide** (pas de bootstrap Drupal). Inconvénient : il faut maintenir un schéma de test aligné avec la base Drupal — et on ne teste pas les interactions Drupal (hooks, `node_load`, etc.).

Cet axe est pertinent quand le moteur a été partiellement découplé et que la couche d'accès aux données utilise du SQL pur plutôt que la Database API Drupal.

### Fixtures pour un moteur de recherche

Les fixtures doivent couvrir les **cas représentatifs** du moteur :

```php
protected function insertSearchFixtures(): void
{
    // Nœuds de différents types
    $this->insertNode(['nid' => 1, 'type' => 'article', 'status' => 1, 'title' => 'PHP Testing']);
    $this->insertNode(['nid' => 2, 'type' => 'article', 'status' => 1, 'title' => 'Drupal Modules']);
    $this->insertNode(['nid' => 3, 'type' => 'page',    'status' => 1, 'title' => 'About Us']);
    $this->insertNode(['nid' => 4, 'type' => 'article', 'status' => 0, 'title' => 'Draft Article']);

    // Taxonomie
    $this->insertTaxonomyIndex(['nid' => 1, 'tid' => 10]); // catégorie "Développement"
    $this->insertTaxonomyIndex(['nid' => 2, 'tid' => 10]);
    $this->insertTaxonomyIndex(['nid' => 3, 'tid' => 20]); // catégorie "Général"

    // Champs custom (field_data_field_date, etc.)
    $this->insertFieldData('field_date', ['entity_id' => 1, 'field_date_value' => '2024-06-15']);
    $this->insertFieldData('field_date', ['entity_id' => 2, 'field_date_value' => '2024-01-10']);
}
```

Quelques règles :

- **Fixtures minimales** : assez de données pour couvrir les cas (filtre actif, inactif, combiné, vide), pas une copie de la prod.
- **Explicites** : chaque fixture est là pour un scénario précis. Commenter pourquoi un nœud est non-publié ou sans taxonomie.
- **Indépendantes** : ne pas compter sur des données laissées par un test précédent. Chaque test (ou classe de tests) charge ses fixtures.

### Assertions sur les résultats

```php
public function testSearchByCategory(): void
{
    $this->insertSearchFixtures();

    $engine = new SearchEngine(static::$connection);
    $results = $engine->search(['category' => 10, 'status' => 'published']);

    // On vérifie les nœuds retournés, pas la requête SQL
    $nids = array_column($results, 'nid');
    sort($nids);
    $this->assertEquals([1, 2], $nids);
}

public function testSearchWithNoResults(): void
{
    $this->insertSearchFixtures();

    $engine = new SearchEngine(static::$connection);
    $results = $engine->search(['category' => 999]);

    $this->assertEmpty($results);
}
```

On vérifie le **résultat** (IDs, nombre, ordre si ORDER BY), pas le texte de la requête SQL. Le test doit rester stable si la requête est refactorisée tant qu'elle retourne les mêmes résultats.

---

## 3. Tests fonctionnels : le pipeline complet

### Quand c'est nécessaire

Quelques tests fonctionnels (ou « end-to-end du module ») sont utiles pour valider que **tout le flux** fonctionne : paramètres HTTP → résolution des filtres → construction de la requête → tables temporaires → exécution → traitement → résultat final. Ils jouent le rôle de tests de fumée (smoke tests) : « le moteur démarre et retourne quelque chose de cohérent ».

### Mise en place dans Drupal 7

Le `DrupalWebTestCase` de Simpletest est l'outil natif. Il installe un site Drupal de test, crée du contenu, et permet de simuler des requêtes HTTP.

```php
class SearchModuleFunctionalTest extends DrupalWebTestCase {

    protected $profile = 'minimal';

    public function setUp() {
        parent::setUp('mymodule_search');
        // Créer du contenu de test
        $this->drupalCreateNode(['type' => 'article', 'title' => 'Test Article']);
    }

    public function testSearchPageReturnsResults() {
        $this->drupalGet('search/custom', ['query' => ['keyword' => 'Test']]);
        $this->assertResponse(200);
        $this->assertText('Test Article');
    }
}
```

C'est le test le plus réaliste, mais aussi le plus lent et le plus fragile : un changement dans le thème, un hook d'un autre module, ou une modification du schéma de la page peuvent casser le test sans régression fonctionnelle réelle.

### Dosage

Réserver les tests fonctionnels aux **scénarios critiques** du moteur :

- La recherche par défaut (sans filtre) retourne bien des résultats.
- Un filtre majeur (catégorie, date) produit bien le résultat attendu.
- Les cas d'erreur (aucun résultat, paramètre invalide) sont gérés correctement.

Trois à cinq tests fonctionnels bien choisis apportent plus de confiance que cinquante tests fragiles.

---

## 4. Tables temporaires : stratégie de test spécifique

### Le défi

Les tables temporaires (`CREATE TEMPORARY TABLE ...`) vivent le temps de la session MySQL. Elles ne sont pas visibles par une autre connexion — donc si le test utilise une connexion séparée pour vérifier le contenu, il ne verra rien.

### Approche recommandée

1. **Tester dans la même connexion** : le test lance le moteur (qui crée et remplit la table temporaire), puis, avant la fin de la transaction, exécute un `SELECT` sur la table temporaire pour vérifier son contenu.

```php
public function testTemporaryTableContainsFilteredNodes(): void
{
    $this->insertSearchFixtures();

    $engine = new SearchEngine(static::$connection);
    // Exécuter la première phase (création de la table temporaire)
    $engine->buildTemporaryResults(['category' => 10]);

    // Vérifier le contenu de la table temporaire (même connexion)
    $rows = static::$connection->query('SELECT nid FROM {search_tmp_results}')
        ->fetchCol();
    sort($rows);
    $this->assertEquals([1, 2], $rows);
}
```

2. **Découper la logique** : si possible, extraire la requête INSERT qui remplit la table temporaire dans une méthode testable séparément. On peut alors tester le SQL du remplissage en intégration et la logique de « quoi mettre dans la table » en unitaire.

3. **Nommer les tables temporaires de manière prévisible** : si le moteur génère des noms dynamiques (`search_tmp_{session_id}`), le rendre configurable ou exposer le nom pour les tests.

---

## 5. `hook_query_alter` et surcharges : comment les tester

Les `hook_query_alter` sont un mécanisme Drupal où d'autres modules modifient une requête après sa construction. Pour le moteur de recherche, cela signifie que la requête testée en intégration peut être différente de celle exécutée en production (si un autre module ajoute des conditions).

### Stratégies

- **Tester avec les hooks actifs** (test fonctionnel ou d'intégration avec les modules activés) : le test reflète la réalité. Inconvénient : les tests du moteur dépendent d'autres modules.
- **Tester sans les hooks** (test d'intégration du module seul) : on vérifie que le moteur produit le bon résultat en isolation. Les interactions avec les autres modules sont testées séparément.
- **Tagger les requêtes** : Drupal 7 permet d'ajouter des tags aux requêtes (`$query->addTag('mymodule_search')`). Les `hook_query_alter` filtrent souvent par tag. En test, on peut vérifier que le tag est bien ajouté, et tester le `hook_query_alter` séparément avec une requête simulée.

Le compromis courant : tester le moteur en isolation (intégration, module seul) et ajouter quelques tests fonctionnels avec les modules actifs pour détecter les interactions non prévues.

---

## 6. Résumé des stratégies

| Couche                      | Stratégie                                        | Outil                     |
|-----------------------------|--------------------------------------------------|---------------------------|
| Logique de filtrage         | Unitaire : entrée (params) → sortie (filtres)   | PHPUnit, sans bootstrap   |
| Traitement des résultats    | Unitaire : entrée (lignes) → sortie (résultats) | PHPUnit, sans bootstrap   |
| Construction de la requête  | Intégration : fixtures + exécution + assertions  | PHPUnit + MySQL           |
| Tables temporaires          | Intégration : même connexion, SELECT intermédiaire | PHPUnit + MySQL        |
| Pipeline complet            | Fonctionnel : bootstrap Drupal + scénarios clés  | DrupalWebTestCase ou PHPUnit + bootstrap |
| Hooks / surcharges          | Fonctionnel : modules activés + scénarios ciblés  | DrupalWebTestCase        |

---

## Prochaines étapes

- **[Exemples concrets](./04-exemples-concrets.md)** : code PHP complet pour chaque stratégie
- **[Mise en pratique](./05-mise-en-pratique.md)** : évaluer l'approche actuelle, plan d'amélioration, CI

---

**Ressources** :

* [PHPUnit — Test Doubles](https://docs.phpunit.de/en/10.5/test-doubles.html)
* [Drupal 7 — Database API](https://www.drupal.org/docs/7/api/database-api)
* [Drupal 7 — Simpletest](https://www.drupal.org/docs/7/testing/simpletest)
