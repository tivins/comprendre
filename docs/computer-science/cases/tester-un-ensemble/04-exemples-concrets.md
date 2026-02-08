# Exemples concrets

Ce chapitre illustre chaque stratégie de test sur un **moteur de recherche Drupal 7** réaliste. Le moteur cherche des nœuds (`node`) avec des filtres dynamiques (type de contenu, taxonomie, date, texte libre), utilise une table temporaire pour les résultats intermédiaires, et applique un traitement de scoring/tri sur les résultats.

Les exemples sont en PHP avec PHPUnit. Les classes sont simplifiées mais représentatives de la structure qu'on peut extraire d'un module Drupal 7.

---

## Exemple 1 : Test unitaire — Résolution des filtres

### Contexte

Le moteur reçoit un tableau associatif de paramètres (issus de la requête HTTP). Une classe `SearchFilterResolver` détermine quels filtres sont actifs, applique les dépendances (le filtre « sous-catégorie » n'est actif que si « catégorie » est défini) et les valeurs par défaut.

### Code sous test

```php
class SearchFilterResolver
{
    public function resolve(array $params): SearchFilters
    {
        $filters = new SearchFilters();

        // Par défaut, on ne cherche que les nœuds publiés
        $filters->add('status', $params['status'] ?? 'published');

        // Type de contenu
        if (!empty($params['content_type'])) {
            $filters->add('content_type', $params['content_type']);
        }

        // Catégorie (taxonomie)
        if (!empty($params['category'])) {
            $filters->add('category', (int) $params['category']);

            // Sous-catégorie : active seulement si la catégorie est définie
            if (!empty($params['subcategory'])) {
                $filters->add('subcategory', (int) $params['subcategory']);
            }
        }

        // Plage de dates
        if (!empty($params['date_from'])) {
            $filters->add('date_from', $params['date_from']);
        }
        if (!empty($params['date_to'])) {
            $filters->add('date_to', $params['date_to']);
        }

        // Texte libre
        if (!empty($params['keyword']) && mb_strlen($params['keyword']) >= 3) {
            $filters->add('keyword', $params['keyword']);
        }

        return $filters;
    }
}
```

### Tests unitaires

```php
class SearchFilterResolverTest extends \PHPUnit\Framework\TestCase
{
    private SearchFilterResolver $resolver;

    protected function setUp(): void
    {
        $this->resolver = new SearchFilterResolver();
    }

    public function testDefaultFiltersWhenNoParams(): void
    {
        $filters = $this->resolver->resolve([]);

        $this->assertTrue($filters->has('status'));
        $this->assertEquals('published', $filters->get('status'));
        $this->assertFalse($filters->has('category'));
        $this->assertFalse($filters->has('keyword'));
    }

    public function testCategoryFilterIsApplied(): void
    {
        $filters = $this->resolver->resolve(['category' => '42']);

        $this->assertTrue($filters->has('category'));
        $this->assertSame(42, $filters->get('category'));
    }

    public function testSubcategoryIgnoredWithoutCategory(): void
    {
        // Sous-catégorie sans catégorie : ignorée (dépendance)
        $filters = $this->resolver->resolve(['subcategory' => '7']);

        $this->assertFalse($filters->has('subcategory'));
    }

    public function testSubcategoryActiveWithCategory(): void
    {
        $filters = $this->resolver->resolve(['category' => '42', 'subcategory' => '7']);

        $this->assertTrue($filters->has('category'));
        $this->assertTrue($filters->has('subcategory'));
        $this->assertSame(7, $filters->get('subcategory'));
    }

    public function testKeywordIgnoredIfTooShort(): void
    {
        $filters = $this->resolver->resolve(['keyword' => 'ab']);

        $this->assertFalse($filters->has('keyword'));
    }

    public function testKeywordAppliedWhenLongEnough(): void
    {
        $filters = $this->resolver->resolve(['keyword' => 'drupal']);

        $this->assertTrue($filters->has('keyword'));
        $this->assertEquals('drupal', $filters->get('keyword'));
    }

    public function testCombinedFilters(): void
    {
        $filters = $this->resolver->resolve([
            'content_type' => 'article',
            'category'     => '10',
            'date_from'    => '2024-01-01',
            'keyword'      => 'testing',
        ]);

        $this->assertTrue($filters->has('content_type'));
        $this->assertTrue($filters->has('category'));
        $this->assertTrue($filters->has('date_from'));
        $this->assertTrue($filters->has('keyword'));
        $this->assertFalse($filters->has('date_to'));
    }
}
```

**Ce qu'on remarque** : pas de BDD, pas de mock, pas de Drupal. Chaque test vérifie une règle de la logique de filtrage. Si un test échoue, on sait exactement quelle règle a régressé. Rapide (millisecondes).

---

## Exemple 2 : Test unitaire — Traitement des résultats (scoring)

### Contexte

Le moteur retourne des lignes brutes (nid, score). Un `SearchResultProcessor` applique un seuil, trie par score décroissant et limite les résultats.

### Code sous test

```php
class SearchResultProcessor
{
    public function process(array $rawRows, float $minScore, int $limit): array
    {
        // Filtrer par score minimum
        $filtered = array_filter($rawRows, fn($row) => $row['score'] >= $minScore);

        // Trier par score décroissant
        usort($filtered, fn($a, $b) => $b['score'] <=> $a['score']);

        // Limiter
        return array_slice($filtered, 0, $limit);
    }
}
```

### Tests unitaires

```php
class SearchResultProcessorTest extends \PHPUnit\Framework\TestCase
{
    private SearchResultProcessor $processor;

    protected function setUp(): void
    {
        $this->processor = new SearchResultProcessor();
    }

    public function testFiltersOutLowScores(): void
    {
        $rows = [
            ['nid' => 1, 'score' => 0.9],
            ['nid' => 2, 'score' => 0.3],
            ['nid' => 3, 'score' => 0.7],
        ];

        $result = $this->processor->process($rows, 0.5, 10);

        $this->assertCount(2, $result);
        $nids = array_column($result, 'nid');
        $this->assertContains(1, $nids);
        $this->assertContains(3, $nids);
        $this->assertNotContains(2, $nids);
    }

    public function testSortsByScoreDescending(): void
    {
        $rows = [
            ['nid' => 1, 'score' => 0.5],
            ['nid' => 2, 'score' => 0.9],
            ['nid' => 3, 'score' => 0.7],
        ];

        $result = $this->processor->process($rows, 0.0, 10);

        $this->assertEquals(2, $result[0]['nid']);
        $this->assertEquals(3, $result[1]['nid']);
        $this->assertEquals(1, $result[2]['nid']);
    }

    public function testLimitsResults(): void
    {
        $rows = [
            ['nid' => 1, 'score' => 0.9],
            ['nid' => 2, 'score' => 0.8],
            ['nid' => 3, 'score' => 0.7],
            ['nid' => 4, 'score' => 0.6],
        ];

        $result = $this->processor->process($rows, 0.0, 2);

        $this->assertCount(2, $result);
    }

    public function testEmptyInputReturnsEmpty(): void
    {
        $result = $this->processor->process([], 0.5, 10);

        $this->assertEmpty($result);
    }

    public function testAllFilteredOutReturnsEmpty(): void
    {
        $rows = [
            ['nid' => 1, 'score' => 0.1],
            ['nid' => 2, 'score' => 0.2],
        ];

        $result = $this->processor->process($rows, 0.5, 10);

        $this->assertEmpty($result);
    }
}
```

**Ce qu'on remarque** : aucune requête SQL. On vérifie trois responsabilités distinctes (filtrage, tri, limite) avec des jeux de données contrôlés. Les cas limites (vide, tout filtré) sont explicitement testés.

---

## Exemple 3 : Test d'intégration — Recherche avec filtres en base

### Contexte

On teste le moteur **de bout en bout de la couche données** : fixtures en base, exécution de la recherche, vérification des résultats. Le bootstrap Drupal est partiel (Database API uniquement).

### Base de test et fixtures

```php
abstract class SearchIntegrationTestCase extends \PHPUnit\Framework\TestCase
{
    protected static \PDO $pdo;

    public static function setUpBeforeClass(): void
    {
        // Connexion à la base de test (MySQL locale ou conteneur)
        static::$pdo = new \PDO(
            'mysql:host=127.0.0.1;dbname=drupal_test;charset=utf8',
            'root', 'root'
        );
        static::$pdo->setAttribute(\PDO::ATTR_ERRMODE, \PDO::ERRMODE_EXCEPTION);
    }

    protected function setUp(): void
    {
        static::$pdo->beginTransaction();
        $this->insertFixtures();
    }

    protected function tearDown(): void
    {
        static::$pdo->rollBack();
    }

    protected function insertFixtures(): void
    {
        // Nœuds
        $this->insertNode(1, 'article', 1, 'Introduction aux tests PHP');
        $this->insertNode(2, 'article', 1, 'Architecture Drupal 7');
        $this->insertNode(3, 'page',    1, 'À propos');
        $this->insertNode(4, 'article', 0, 'Brouillon tests');   // non-publié

        // Taxonomie
        $this->insertTaxonomyIndex(1, 10); // nœud 1 → catégorie "Développement"
        $this->insertTaxonomyIndex(2, 10); // nœud 2 → catégorie "Développement"
        $this->insertTaxonomyIndex(3, 20); // nœud 3 → catégorie "Général"

        // Dates (champ custom)
        $this->insertFieldDate(1, '2024-06-15');
        $this->insertFieldDate(2, '2024-01-10');
        $this->insertFieldDate(3, '2023-12-01');
    }

    private function insertNode(int $nid, string $type, int $status, string $title): void
    {
        static::$pdo->prepare(
            'INSERT INTO node (nid, type, status, title) VALUES (?, ?, ?, ?)'
        )->execute([$nid, $type, $status, $title]);
    }

    private function insertTaxonomyIndex(int $nid, int $tid): void
    {
        static::$pdo->prepare(
            'INSERT INTO taxonomy_index (nid, tid) VALUES (?, ?)'
        )->execute([$nid, $tid]);
    }

    private function insertFieldDate(int $entityId, string $value): void
    {
        static::$pdo->prepare(
            'INSERT INTO field_data_field_date (entity_id, field_date_value) VALUES (?, ?)'
        )->execute([$entityId, $value]);
    }
}
```

### Tests d'intégration

```php
class SearchEngineIntegrationTest extends SearchIntegrationTestCase
{
    private SearchEngine $engine;

    protected function setUp(): void
    {
        parent::setUp();
        $this->engine = new SearchEngine(static::$pdo);
    }

    public function testSearchByCategoryReturnsMatchingPublishedNodes(): void
    {
        $results = $this->engine->search(['category' => 10]);

        $nids = array_column($results, 'nid');
        sort($nids);
        $this->assertEquals([1, 2], $nids);
        // Le nœud 3 (catégorie 20) et le 4 (non-publié) ne sont pas retournés
    }

    public function testSearchByContentType(): void
    {
        $results = $this->engine->search(['content_type' => 'page']);

        $this->assertCount(1, $results);
        $this->assertEquals(3, $results[0]['nid']);
    }

    public function testSearchByDateRange(): void
    {
        $results = $this->engine->search([
            'date_from' => '2024-01-01',
            'date_to'   => '2024-12-31',
        ]);

        $nids = array_column($results, 'nid');
        sort($nids);
        // Nœuds 1 et 2 sont dans la plage ; nœud 3 est en 2023 ; nœud 4 est non-publié
        $this->assertEquals([1, 2], $nids);
    }

    public function testCombinedFilters(): void
    {
        $results = $this->engine->search([
            'content_type' => 'article',
            'category'     => 10,
            'date_from'    => '2024-06-01',
        ]);

        // Seul le nœud 1 (article, catégorie 10, date 2024-06-15, publié)
        $this->assertCount(1, $results);
        $this->assertEquals(1, $results[0]['nid']);
    }

    public function testSearchWithNoMatchReturnsEmpty(): void
    {
        $results = $this->engine->search(['category' => 999]);

        $this->assertEmpty($results);
    }

    public function testUnpublishedNodesAreExcludedByDefault(): void
    {
        $results = $this->engine->search([]); // pas de filtre

        $nids = array_column($results, 'nid');
        $this->assertNotContains(4, $nids, 'Le nœud non-publié ne doit pas apparaître');
    }
}
```

**Ce qu'on remarque** :

- Chaque test insère les mêmes fixtures (via `setUp` → transaction), puis exécute la recherche et vérifie le résultat.
- Le rollback en `tearDown` garantit l'isolation.
- Les assertions portent sur les **IDs et le nombre** de résultats, pas sur le SQL.
- Les cas combinés et les cas vides sont couverts.

---

## Exemple 4 : Test d'intégration — Table temporaire

### Contexte

Le moteur crée une table temporaire `search_tmp_nids` pour stocker les IDs pré-filtrés, puis joint cette table dans la requête finale (scoring, tri).

### Test

```php
class SearchTempTableIntegrationTest extends SearchIntegrationTestCase
{
    public function testTempTableContainsPreFilteredNids(): void
    {
        $engine = new SearchEngine(static::$pdo);

        // Phase 1 : le moteur crée et remplit la table temporaire
        $engine->buildPreFilteredResults(['category' => 10, 'status' => 'published']);

        // Vérifier le contenu de la table temporaire (même connexion PDO)
        $stmt = static::$pdo->query('SELECT nid FROM search_tmp_nids ORDER BY nid');
        $nids = $stmt->fetchAll(\PDO::FETCH_COLUMN);

        $this->assertEquals([1, 2], $nids);
    }

    public function testTempTableIsEmptyWhenNoMatch(): void
    {
        $engine = new SearchEngine(static::$pdo);
        $engine->buildPreFilteredResults(['category' => 999]);

        $stmt = static::$pdo->query('SELECT nid FROM search_tmp_nids');
        $nids = $stmt->fetchAll(\PDO::FETCH_COLUMN);

        $this->assertEmpty($nids);
    }
}
```

Le point clé : le `SELECT` sur la table temporaire est exécuté **dans la même connexion PDO** que l'appel au moteur — sinon la table n'est pas visible.

---

## Exemple 5 : Test du `SearchQueryBuilder` via introspection

### Contexte

Si vous avez extrait un `SearchQueryBuilder` qui construit un objet `SelectQuery` Drupal, vous pouvez inspecter la requête construite sans l'exécuter. Attention : cette approche est **plus fragile** (elle dépend de la structure interne de l'objet `SelectQuery`), mais utile pour vérifier que les bons JOINs et conditions sont générés.

### Test

```php
class SearchQueryBuilderTest extends \PHPUnit\Framework\TestCase
{
    public function testBuildWithCategoryAddsJoinAndCondition(): void
    {
        // En supposant que le builder est découplé de l'exécution
        $builder = new SearchQueryBuilder();
        $filters = new SearchFilters();
        $filters->add('category', 42);
        $filters->add('status', 'published');

        $query = $builder->build($filters);

        // Convertir en chaîne (Drupal SelectQuery implémente __toString)
        $sql = (string) $query;

        $this->assertStringContainsString('taxonomy_index', $sql);
        $this->assertStringContainsString('tid', $sql);
        $this->assertStringContainsString('status', $sql);
    }
}
```

**Limite** : tester le texte SQL est fragile (un changement d'alias, d'ordre des JOINs, de formatage casse le test). Préférer les tests d'intégration (exécution réelle) sauf si vous avez besoin de vérifier qu'un filtre spécifique ajoute bien une jointure sans exécuter la requête.

---

## Récapitulatif par exemple

| # | Niveau       | Ce qu'on vérifie                                    | BDD nécessaire ? |
|---|-------------|-----------------------------------------------------|-------------------|
| 1 | Unitaire    | Résolution des filtres (logique de combinaison)      | Non               |
| 2 | Unitaire    | Traitement des résultats (scoring, tri, limite)      | Non               |
| 3 | Intégration | Recherche complète avec filtres (résultats en base)  | Oui               |
| 4 | Intégration | Contenu de la table temporaire                       | Oui               |
| 5 | Unitaire*   | Structure de la requête construite (introspection)   | Non               |

\* L'exemple 5 est unitaire en termes d'exécution (pas de BDD) mais fragile en termes de maintenance.

---

## Prochaines étapes

**[Mise en pratique](./05-mise-en-pratique.md)** : évaluer l'approche actuelle, plan d'amélioration progressif, intégration dans le workflow et la CI, pièges à éviter.
