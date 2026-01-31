# Exemples concrets de types de tests

Ce chapitre illustre les différents types de tests sur des cas réalistes : logique métier, service avec repository, API HTTP, et un aperçu E2E. Les exemples sont en pseudo-code style Java/TypeScript pour rester lisibles dans plusieurs écosystèmes.

## Exemple 1 : Test unitaire — logique de réduction

### Contexte

Un `PriceCalculator` applique une réduction selon un code promo. La règle : 10 % si code `PROMO10`, 20 % si `PROMO20`, sinon pas de réduction. On veut vérifier la logique sans BDD ni réseau.

### Code sous test (simplifié)

```java
class PriceCalculator {
    double applyDiscount(double amount, String promoCode) {
        if (promoCode == null || promoCode.isEmpty()) return amount;
        return switch (promoCode) {
            case "PROMO10" -> amount * 0.9;
            case "PROMO20" -> amount * 0.8;
            default -> amount;
        };
    }
}
```

### Tests unitaires

```java
@Test
void applyDiscount_PROMO10_reduit_de_10_pourcent() {
    PriceCalculator calc = new PriceCalculator();
    assertEquals(90.0, calc.applyDiscount(100.0, "PROMO10"), 0.01);
}

@Test
void applyDiscount_code_inconnu_ne_modifie_pas_le_prix() {
    PriceCalculator calc = new PriceCalculator();
    assertEquals(100.0, calc.applyDiscount(100.0, "INVALID"), 0.01);
}

@Test
void applyDiscount_code_null_retourne_le_montant_brut() {
    PriceCalculator calc = new PriceCalculator();
    assertEquals(100.0, calc.applyDiscount(100.0, null), 0.01);
}
```

Rapides, sans dépendance externe, un échec pointe directement la règle en cause.

---

## Exemple 2 : Test unitaire avec mock — service et repository

### Contexte

Un `OrderService` place une commande en appelant un `OrderRepository` et un `NotificationService`. On veut vérifier que le service appelle le repository avec les bons arguments et envoie une notification, sans toucher à une vraie BDD.

### Code sous test (simplifié)

```java
class OrderService {
    private final OrderRepository orderRepository;
    private final NotificationService notificationService;

    public OrderService(OrderRepository orderRepository, NotificationService notificationService) {
        this.orderRepository = orderRepository;
        this.notificationService = notificationService;
    }

    String placeOrder(Order order) {
        String id = orderRepository.save(order);
        notificationService.sendOrderConfirmation(order, id);
        return id;
    }
}
```

### Test unitaire avec mocks

```java
@Test
void placeOrder_sauvegarde_et_envoie_une_notification() {
    OrderRepository mockRepo = mock(OrderRepository.class);
    NotificationService mockNotif = mock(NotificationService.class);
    when(mockRepo.save(any(Order.class))).thenReturn("order-123");

    OrderService service = new OrderService(mockRepo, mockNotif);
    Order order = new Order("user-1", List.of("item-A"));
    String id = service.placeOrder(order);

    assertEquals("order-123", id);
    verify(mockRepo).save(order);
    verify(mockNotif).sendOrderConfirmation(order, "order-123");
}
```

On vérifie le **comportement** du service (appels aux collaborateurs), pas la persistance réelle. La persistance sera couverte par un test d’intégration.

---

## Exemple 3 : Test d’intégration — service + repository + BDD

### Contexte

Même `OrderService` et un `JdbcOrderRepository` qui écrit en base. On veut s’assurer que la commande est bien persistée et relue (schéma, mapping, contraintes). BDD de test : H2, Postgres en conteneur, ou équivalent.

### Test d’intégration

```java
@Test
void placeOrder_persiste_et_retourne_l_id() {
    DataSource ds = createTestDataSource(); // H2 ou Testcontainers
    OrderRepository repo = new JdbcOrderRepository(ds);
    NotificationService mockNotif = mock(NotificationService.class);
    OrderService service = new OrderService(repo, mockNotif);

    Order order = new Order("user-1", List.of("item-A"));
    String id = service.placeOrder(order);

    assertNotNull(id);
    Order saved = repo.findById(id);
    assertNotNull(saved);
    assertEquals("user-1", saved.getUserId());
}
```

Ici on ne mocke pas le repository : on vérifie l’intégration service + BDD. Plus lent qu’un unitaire, à garder en nombre raisonnable.

---

## Exemple 4 : Test d’intégration — API HTTP

### Contexte

Un contrôleur REST expose `POST /orders`. On veut vérifier que la requête HTTP aboutit à une création en BDD et renvoie le bon statut et corps. Pas de navigateur : client HTTP (RestAssured, WebTestClient, fetch, etc.) contre l’application (ou un slice avec le contrôleur seul selon le framework).

### Test (idée, style RestAssured / WebTestClient)

```java
@Test
void post_orders_creates_order_and_returns_201() {
    given()
        .contentType(JSON)
        .body("{\"userId\":\"user-1\",\"items\":[\"item-A\"]}")
    .when()
        .post("/orders")
    .then()
        .statusCode(201)
        .body("id", notNullValue())
        .body("userId", equalTo("user-1"));
}
```

C’est un test d’intégration : plusieurs couches (contrôleur, service, repository, BDD) travaillent ensemble. À distinguer d’un test E2E qui passerait par l’interface utilisateur réelle.

---

## Exemple 5 : Test E2E — parcours utilisateur (aperçu)

### Contexte

On veut s’assurer qu’un utilisateur peut se connecter et passer une commande via l’interface web. Outil type Playwright ou Selenium.

### Test (idée, style Playwright)

```typescript
test('connexion puis commande', async ({ page }) => {
  await page.goto('/login');
  await page.fill('#email', 'user@example.com');
  await page.fill('#password', 'secret');
  await page.click('button[type=submit]');
  await expect(page).toHaveURL(/\/dashboard/);

  await page.goto('/cart');
  await page.click('button:has-text("Commander")');
  await expect(page.locator('.order-confirmation')).toBeVisible();
});
```

Lent, sensible aux changements d’UI et au timing. À réserver à quelques scénarios critiques et à maintenir soigneusement (sélecteurs stables, données de test dédiées).

---

## Récapitulatif par exemple

| Exemple | Type        | Ce qu’on vérifie                          | Dépendances mockées / réelles      |
|---------|-------------|-------------------------------------------|------------------------------------|
| 1       | Unitaire    | Règle de réduction                        | Aucune                             |
| 2       | Unitaire    | Appels repository + notification          | Repository, notification (mocks)   |
| 3       | Intégration | Persistance réelle en BDD                 | BDD réelle, notification mockée  |
| 4       | Intégration | Endpoint HTTP + création en BDD           | BDD réelle, appli ou slice        |
| 5       | E2E         | Parcours utilisateur complet              | Appli + BDD + navigateur           |

---

## Prochaines étapes

**[Mise en pratique](./04-mise-en-pratique.md)** : quand écrire quel type de test, pièges courants (flakiness, sur-mock, couverture trompeuse), intégration dans le workflow et la CI.
