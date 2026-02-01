# Exemples concrets

Ce chapitre illustre les méthodes recommandées sur des cas réalistes : rapport avec agrégation, requête multi-tables avec JOINs, et traitement métier après fetch. Les exemples sont en pseudo-code inspiré de Java/TypeScript avec des requêtes SQL réalistes.

## Exemple 1 : Rapport avec agrégation — test unitaire du traitement

### Contexte

Un service produit un **rapport de ventes par catégorie** : le repository exécute une requête SQL avec GROUP BY et SUM ; le service reçoit les lignes et applique une règle (ex. masquer les catégories dont le total est inférieur à un seuil). On veut vérifier la logique de filtrage sans BDD.

### Requête (idée)

```sql
SELECT category_id, category_name, SUM(amount) AS total
FROM orders o
JOIN order_lines ol ON o.id = ol.order_id
JOIN products p ON ol.product_id = p.id
JOIN categories c ON p.category_id = c.id
WHERE o.created_at BETWEEN ? AND ?
GROUP BY category_id, category_name
```

### Code sous test (simplifié)

```java
class SalesReportService {
    private final SalesReportRepository repository;

    SalesReportService(SalesReportRepository repository) {
        this.repository = repository;
    }

    List<CategoryTotal> getReport(LocalDate from, LocalDate to, double minTotal) {
        List<CategoryRow> rows = repository.findTotalsByCategory(from, to);
        return rows.stream()
            .filter(row -> row.getTotal() >= minTotal)
            .map(row -> new CategoryTotal(row.getCategoryName(), row.getTotal()))
            .toList();
    }
}
```

### Test unitaire avec mock du repository

```java
@Test
void getReport_filtre_les_categories_sous_le_seuil() {
    SalesReportRepository mockRepo = mock(SalesReportRepository.class);
    List<CategoryRow> rows = List.of(
        new CategoryRow(1, "Électronique", 5000.0),
        new CategoryRow(2, "Accessoires", 200.0),
        new CategoryRow(3, "Livres", 1500.0)
    );
    when(mockRepo.findTotalsByCategory(any(), any())).thenReturn(rows);

    SalesReportService service = new SalesReportService(mockRepo);
    List<CategoryTotal> result = service.getReport(
        LocalDate.of(2024, 1, 1),
        LocalDate.of(2024, 1, 31),
        1000.0
    );

    assertEquals(2, result.size());
    assertTrue(result.stream().anyMatch(r -> "Électronique".equals(r.getCategoryName()) && r.getTotal() == 5000.0));
    assertTrue(result.stream().anyMatch(r -> "Livres".equals(r.getCategoryName()) && r.getTotal() == 1500.0));
    assertFalse(result.stream().anyMatch(r -> "Accessoires".equals(r.getCategoryName())));
}
```

On vérifie le **traitement** (filtrage par seuil) avec des données en entrée fixées ; la requête SQL n’est pas exécutée. La requête elle-même sera couverte par un test d’intégration.

---

## Exemple 2 : Rapport avec agrégation — test d’intégration de la requête

### Contexte

Même rapport : on veut s’assurer que la **requête** retourne les bons totaux pour un jeu de données en base. BDD de test : Testcontainers MySQL (ou MySQL dédiée) avec schéma et fixtures.

### Fixtures (idée)

```sql
-- clients, commandes, lignes, produits, catégories
INSERT INTO categories (id, name) VALUES (1, 'Électronique'), (2, 'Accessoires');
INSERT INTO products (id, category_id, name) VALUES (1, 1, 'Laptop'), (2, 2, 'Souris');
INSERT INTO orders (id, user_id, created_at) VALUES ('ord-1', 'u1', '2024-01-15 10:00:00');
INSERT INTO order_lines (order_id, product_id, quantity, unit_price) VALUES ('ord-1', 1, 2, 2500.0), ('ord-1', 2, 1, 50.0);
```

### Test d’intégration

```java
@Test
void findTotalsByCategory_retourne_les_agregats_attendus() {
    DataSource ds = testDataSource(); // Testcontainers MySQL + schéma + fixtures
    SalesReportRepository repo = new JdbcSalesReportRepository(ds);

    List<CategoryRow> result = repo.findTotalsByCategory(
        LocalDate.of(2024, 1, 1),
        LocalDate.of(2024, 1, 31)
    );

    assertEquals(2, result.size());
    CategoryRow electronics = result.stream().filter(r -> r.getCategoryId() == 1).findFirst().orElseThrow();
    assertEquals(5000.0, electronics.getTotal(), 0.01); // 2 * 2500
    CategoryRow accessories = result.stream().filter(r -> r.getCategoryId() == 2).findFirst().orElseThrow();
    assertEquals(50.0, accessories.getTotal(), 0.01);
}
```

Ici on ne mocke pas le repository : on vérifie que la **requête** + le schéma + les fixtures produisent le bon résultat. Plus lent qu’un unitaire ; à garder en nombre maîtrisé.

---

## Exemple 3 : Requête multi-tables (JOINs) — intégration

### Contexte

Une requête récupère les **commandes avec détails client et lignes** (plusieurs JOINs). On veut valider que les lignes retournées correspondent aux fixtures (bon client, bonnes lignes, pas de doublon ni d’oubli).

### Requête (idée)

```sql
SELECT o.id AS order_id, o.created_at, u.email, ol.product_id, ol.quantity, p.name AS product_name
FROM orders o
JOIN users u ON o.user_id = u.id
JOIN order_lines ol ON o.id = ol.order_id
JOIN products p ON ol.product_id = p.id
WHERE o.id = ?
```

### Test d’intégration

```java
@Test
void findOrderWithDetails_retourne_commande_avec_lignes_et_client() {
    DataSource ds = testDataSource(); // fixtures : 1 commande, 2 lignes, 1 user, 2 produits
    OrderDetailRepository repo = new JdbcOrderDetailRepository(ds);

    OrderDetail result = repo.findOrderWithDetails("ord-1");

    assertNotNull(result);
    assertEquals("ord-1", result.getOrderId());
    assertEquals("user@example.com", result.getUserEmail());
    assertEquals(2, result.getLines().size());
    assertTrue(result.getLines().stream().anyMatch(l -> "Laptop".equals(l.getProductName()) && l.getQuantity() == 2));
    assertTrue(result.getLines().stream().anyMatch(l -> "Souris".equals(l.getProductName()) && l.getQuantity() == 1));
}
```

Les fixtures définissent exactement une commande, un client, deux lignes ; le test vérifie le **mapping** et la **structure** du résultat (pas de ligne en trop ou en moins). L’ordre des lignes peut être normalisé en test (tri par product_id) si la requête n’impose pas d’ORDER BY.

---

## Exemple 4 : Traitement métier après fetch — unitaire

### Contexte

Le repository retourne des **lignes brutes** (ex. statistiques par jour) ; le service calcule des **indicateurs dérivés** (tendance, pourcentage d’évolution). On veut tester ces calculs sans BDD.

### Code sous test (simplifié)

```java
class StatsService {
    List<DailyStats> computeTrends(List<DailyStatRow> rows) {
        if (rows.isEmpty()) return List.of();
        List<DailyStats> result = new ArrayList<>();
        double previous = rows.get(0).getValue();
        for (DailyStatRow row : rows) {
            double variation = previous == 0 ? 0 : (row.getValue() - previous) / previous * 100;
            result.add(new DailyStats(row.getDate(), row.getValue(), variation));
            previous = row.getValue();
        }
        return result;
    }
}
```

### Test unitaire avec données fixées

```java
@Test
void computeTrends_calcule_la_variation_pourcentuelle() {
    StatsService service = new StatsService();
    List<DailyStatRow> rows = List.of(
        new DailyStatRow(LocalDate.of(2024, 1, 1), 100.0),
        new DailyStatRow(LocalDate.of(2024, 1, 2), 120.0),
        new DailyStatRow(LocalDate.of(2024, 1, 3), 90.0)
    );

    List<DailyStats> result = service.computeTrends(rows);

    assertEquals(3, result.size());
    assertEquals(0.0, result.get(0).getVariationPercent(), 0.01);
    assertEquals(20.0, result.get(1).getVariationPercent(), 0.01);
    assertEquals(-25.0, result.get(2).getVariationPercent(), 0.01);
}
```

Aucune requête ni BDD : on teste uniquement la **logique de calcul** sur des lignes fournies en entrée. Rapide et stable.

---

## Récapitulatif par exemple

| Exemple | Niveau        | Ce qu’on vérifie                         | Données                          |
|---------|---------------|------------------------------------------|----------------------------------|
| 1       | Unitaire      | Filtrage par seuil (traitement)          | Mock du repository (lignes fixées) |
| 2       | Intégration   | Agrégation SQL (GROUP BY, SUM)           | Fixtures en BDD                  |
| 3       | Intégration   | JOINs et mapping (commande + lignes)     | Fixtures en BDD                  |
| 4       | Unitaire      | Calcul de tendance (traitement)          | Liste de lignes en dur           |

---

## Prochaines étapes

**[Mise en pratique](./05-mise-en-pratique.md)** : quand privilégier quelle approche, pièges courants (données partagées, ordre, flakiness), intégration dans le workflow et la CI.
