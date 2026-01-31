# Types de tests et pyramide

Ce chapitre définit opérationnellement les principaux types de tests (unitaire, intégration, E2E, contractuels) et présente la pyramide des tests ainsi que ses variantes.

## 1. Tests unitaires

### Définition

Un **test unitaire** exerce une **unité de code isolée** (fonction, méthode, classe) en remplaçant ses dépendances par des mocks ou des stubs. Pas de vraie base de données, pas de réseau, pas de système de fichiers non maîtrisé. L’unité peut être une classe entière si elle est petite et sans dépendances lourdes, ou une seule fonction pure.

En pratique : une classe sous test, des collaborateurs mockés, des assertions sur le retour ou les appels aux mocks.

### Caractéristiques

- **Rapides** : millisecondes par test. Des centaines ou milliers peuvent tourner à chaque changement.
- **Ciblés** : un échec indique précisément quelle unité et quel scénario posent problème.
- **Isolés** : pas d’ordre d’exécution, pas d’état partagé entre tests (idéalement).
- **Limite** : ils ne vérifient pas que les briques s’assemblent correctement (contrats, SQL, API réelles).

### Quand les utiliser

- Logique métier pure (calculs, règles, transformations).
- Comportement d’une classe en interaction avec des dépendances que vous contrôlez via mocks (ex. : « le service appelle bien le repository avec ces arguments »).
- Régressions sur des cas limites (valeurs nulles, listes vides, erreurs).

### Exemple (idée)

```java
// Unité : PriceCalculator.applyDiscount(Order)
@Test
void applyDiscount_reduit_le_total_selon_le_taux() {
    PriceCalculator calc = new PriceCalculator();
    Order order = new Order(100.0, "PROMO10");
    double result = calc.applyDiscount(order);
    assertEquals(90.0, result, 0.01);
}
```

Pas de BDD ni de service externe : uniquement la logique de réduction.

---

## 2. Tests d’intégration

### Définition

Un **test d’intégration** exerce **plusieurs composants ensemble** dans un environnement proche de la réalité : par exemple un service + un repository + une vraie base (ou un conteneur type Testcontainers). On vérifie que les pièces collent (requêtes SQL, mapping, transactions, appels entre modules).

Les dépendances externes (API tierces, messagerie) sont souvent mockées ou remplacées par des doubles pour garder le test maîtrisable.

### Caractéristiques

- **Plus lents** : secondes par test. On en garde un nombre raisonnable (dizaines à quelques centaines selon le projet).
- **Plus fragiles** : configuration BDD, réseau, ordre d’exécution peuvent introduire du flakiness si mal gérés.
- **Utilité** : confirment que les contrats (interface repository, schéma BDD, API interne) sont respectés.

### Quand les utiliser

- Vérifier qu’un service persiste et relit correctement en BDD.
- Tester un endpoint HTTP (controller + service + repository) avec une BDD réelle ou en mémoire.
- Valider des requêtes SQL, des transactions, des contraintes.

### Exemple (idée)

```java
// Intégration : OrderService + JdbcOrderRepository + H2/Postgres
@Test
void placeOrder_persiste_la_commande_et_retourne_l_id() {
    OrderRepository repo = new JdbcOrderRepository(dataSource); // vraie BDD de test
    OrderService service = new OrderService(repo, mockNotificationService);
    Order order = new Order("user-1", List.of("item-A"));
    String id = service.placeOrder(order);
    assertNotNull(id);
    Order saved = repo.findById(id);
    assertEquals("user-1", saved.getUserId());
}
```

Ici on ne mocke pas le repository : on vérifie l’intégration service + BDD.

---

## 3. Tests bout en bout (E2E)

### Définition

Un **test E2E** (end-to-end) simule un **parcours utilisateur complet** via l’interface réelle : navigateur (Playwright, Selenium) ou client HTTP sur l’API publique. L’application tourne (ou une instance dédiée) avec une BDD de test ; les actions (clics, formulaires, appels API) sont automatisées et les résultats vérifiés.

### Caractéristiques

- **Les plus lents et les plus coûteux** : démarrage appli, BDD, réseau. Souvent réservés à la CI ou à des runs périodiques.
- **Les plus sensibles** : timing, sélecteurs UI, données partagées peuvent rendre les tests flaky.
- **Apport** : confiance que le flux métier critique fonctionne pour l’utilisateur final.

### Quand les utiliser

- Quelques scénarios critiques (connexion, commande, inscription) pour sécuriser les releases.
- En complément des tests unitaires et d’intégration, pas en remplacement — le ratio coût/bénéfice décroît vite si on multiplie les E2E.

### Exemple (idée)

```typescript
// E2E : parcours utilisateur dans le navigateur
test('utilisateur peut passer une commande', async ({ page }) => {
  await page.goto('/login');
  await page.fill('#email', 'user@example.com');
  await page.fill('#password', 'secret');
  await page.click('button[type=submit]');
  await page.goto('/cart');
  await page.click('button:has-text("Commander")');
  await expect(page.locator('.order-confirmation')).toBeVisible();
});
```

---

## 4. Tests contractuels (et composant)

### Définition

Les **tests contractuels** vérifient qu’un **contrat d’interface** est respecté entre deux parties (client/serveur, front/back, producteur/consommateur de messages). Souvent utilisés en microservices : un consumer vérifie qu’il comprend les réponses du provider (Pact, Spring Cloud Contract, etc.).

Les **tests de composant** (au sens « component test ») exercent un composant (ex. une API ou un service) dans un environnement proche de la prod, en remplaçant certains partenaires par des stubs. La frontière avec l’intégration est floue ; l’important est de savoir ce qu’on isole et ce qu’on vérifie.

### Quand les utiliser

- Plusieurs équipes ou services qui évoluent indépendamment : le contrat évite les régressions silencieuses à l’interface.
- Front et back découplés : s’assurer que les réponses API correspondent à ce que le front attend.

---

## Pyramide des tests

La **pyramide** (Mike Cohn, popularisée par Martin Fowler) résume l’idée suivante :

- **Base large** : beaucoup de tests unitaires (rapides, stables, ciblés).
- **Milieu** : moins de tests d’intégration (plus lents, vérifient l’assemblage).
- **Sommet** : peu de tests E2E (lents, fragiles, scénarios critiques).

Objectif : feedback rapide au quotidien, confiance sur l’ensemble sans exploser le temps de build.

### Variantes

- **Cône inversé (ice cream cone)** : beaucoup d’E2E, peu d’unitaires. Souvent considéré comme anti-pattern (lent, fragile, difficile à maintenir).
- **Diamant** : peu d’unitaires, beaucoup d’intégration au centre, peu d’E2E. Certaines équipes privilégient l’intégration pour des systèmes où l’unitaire apporte moins (ex. logique très couplée à l’infra).
- **Pyramide** reste une référence pragmatique : adapter les proportions au contexte (domaine, stack, tolérance au flakiness).

Dans votre contexte : combien de temps le pipeline peut-il prendre ? Combien de scénarios E2E sont vraiment indispensables ? La réponse guide le sommet de la pyramide.

---

## Synthèse rapide

| Type        | Périmètre              | Vitesse   | Nombre conseillé | Question typique                          |
|------------|-------------------------|-----------|------------------|-------------------------------------------|
| Unitaire   | Une unité, mocks        | Rapide    | Élevé            | Cette logique calcule-t-elle juste ?      |
| Intégration| Plusieurs composants    | Moyen     | Modéré           | Ces briques fonctionnent-elles ensemble ? |
| E2E        | Parcours utilisateur    | Lent      | Faible           | L’utilisateur peut-il faire X ?           |
| Contractuel| Interface entre parties | Variable  | Ciblé            | Le contrat API/message est-il respecté ?  |

---

## Prochaines étapes

- **[Exemples concrets](./03-exemples-concrets.md)** : services métier, API, BDD, front — mêmes idées en cas concrets.
- **[Mise en pratique](./04-mise-en-pratique.md)** : quand écrire quoi, pièges à éviter, intégration dans le workflow.

---

**Ressource** : [Martin Fowler — The Practical Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html)
