# Introduction à l'injection de dépendances

## Qu'est-ce que l'injection de dépendances ?

L'**injection de dépendances** (Dependency Injection, DI) est une technique selon laquelle un objet **reçoit** les autres objets dont il a besoin (ses dépendances), au lieu de les créer lui-même ou de les récupérer via une factory ou un singleton global.

En résumé : **« Ne me crée pas mes dépendances, donne-les-moi. »**

### Définition formelle

> L'injection de dépendances consiste à passer (injecter) les dépendances d'une classe depuis l'extérieur, plutôt que de les instancier ou les résoudre à l'intérieur de la classe.

Une **dépendance** est tout objet dont une classe a besoin pour fonctionner : un repository, un service, un client HTTP, un logger, etc.

## Le problème : le couplage fort

### Exemple sans injection de dépendances

Imaginons une classe `OrderService` qui doit enregistrer une commande et envoyer un email de confirmation.

```java
class OrderService {
    private EmailService emailService;
    private OrderRepository orderRepository;

    public OrderService() {
        this.emailService = new SmtpEmailService();      // Création directe
        this.orderRepository = new JdbcOrderRepository(); // Création directe
    }

    public void placeOrder(Order order) {
        orderRepository.save(order);
        emailService.send(order.getCustomerEmail(), "Commande confirmée", "...");
    }
}
```

**Problèmes** :

1. **Couplage fort** : `OrderService` dépend directement de `SmtpEmailService` et de `JdbcOrderRepository`. Changer d'implémentation (ex. SendGrid, MongoDB) impose de modifier `OrderService`.
2. **Difficile à tester** : Pour tester `OrderService`, on ne peut pas remplacer l'email ou la base par des mocks sans modifier le code de production.
3. **Violation du DIP** : Le module de haut niveau (`OrderService`) dépend des modules de bas niveau (implémentations concrètes).
4. **Rigidité** : Ajouter une nouvelle option (ex. notification SMS) ou une variante (ex. email en mode test) complique encore la classe.

### Variante tout aussi problématique : le Service Locator

Parfois, au lieu de créer les dépendances dans le constructeur, on les récupère via un « service locator » (registre global) :

```java
class OrderService {
    private EmailService emailService;
    private OrderRepository orderRepository;

    public OrderService() {
        this.emailService = ServiceLocator.get(EmailService.class);
        this.orderRepository = ServiceLocator.get(OrderRepository.class);
    }

    public void placeOrder(Order order) {
        orderRepository.save(order);
        emailService.send(order.getCustomerEmail(), "Commande confirmée", "...");
    }
}
```

**Problèmes** : La classe dépend toujours d'un détail d'infrastructure (le `ServiceLocator`), et les dépendances restent cachées : on ne voit pas en lisant la signature de `OrderService` qu'il a besoin de `EmailService` et `OrderRepository`. Les tests doivent aussi configurer le `ServiceLocator`, ce qui complique le setup.

## La solution : injecter les dépendances

Au lieu que `OrderService` crée ou récupère ses dépendances, **on les lui passe** depuis l'extérieur (par le constructeur, par exemple).

```java
interface EmailService {
    void send(String to, String subject, String body);
}

interface OrderRepository {
    void save(Order order);
}

class OrderService {
    private final EmailService emailService;
    private final OrderRepository orderRepository;

    public OrderService(EmailService emailService, OrderRepository orderRepository) {
        this.emailService = emailService;
        this.orderRepository = orderRepository;
    }

    public void placeOrder(Order order) {
        orderRepository.save(order);
        emailService.send(order.getCustomerEmail(), "Commande confirmée", "...");
    }
}
```

**Création et assemblage** (souvent dans une couche « composition root » ou un conteneur IoC) :

```java
// Quelque part à la racine de l'application
EmailService emailService = new SmtpEmailService();
OrderRepository orderRepository = new JdbcOrderRepository(connection);
OrderService orderService = new OrderService(emailService, orderRepository);
```

**Avantages** :

1. **Découplage** : `OrderService` ne connaît que les interfaces `EmailService` et `OrderRepository`. On peut fournir n'importe quelle implémentation (Smtp, SendGrid, Mock, etc.).
2. **Testabilité** : Dans les tests, on injecte des mocks ou des stubs :
   ```java
   OrderService service = new OrderService(mockEmailService, mockOrderRepository);
   service.placeOrder(order);
   // Vérifier que mockEmailService.send() a été appelé
   ```
3. **Dépendances explicites** : En regardant le constructeur, on voit immédiatement de quoi dépend `OrderService`.
4. **Respect du DIP** : Le module de haut niveau dépend des abstractions (interfaces), pas des implémentations.

## Relation avec le principe DIP (SOLID)

Le **principe d'inversion des dépendances** (Dependency Inversion Principle) dit :

- Les modules de haut niveau ne doivent pas dépendre des modules de bas niveau ; les deux doivent dépendre d'abstractions.
- Les abstractions ne doivent pas dépendre des détails ; les détails doivent dépendre des abstractions.

L'injection de dépendances est **la technique** qui permet d'appliquer ce principe en pratique :

- On définit des **abstractions** (interfaces) pour les dépendances.
- Les classes de haut niveau (services, orchestrateurs) reçoivent ces abstractions par injection.
- Les implémentations concrètes (SmtpEmailService, JdbcOrderRepository) sont créées ailleurs et injectées.

Sans DI, une classe est tentée de faire `new ConcreteImplementation()`, ce qui crée une dépendance directe vers le détail. Avec DI, elle ne reçoit que l'abstraction et reste indépendante de l'implémentation réelle.

## Analogie du quotidien

Pensez à un **cuisinier** :

- **Sans DI** : Le cuisinier possède sa propre ferme, son propre moulin et son propre four. Il dépend totalement de ces installations. Pour changer de fournisseur de farine, il doit tout réorganiser.
- **Avec DI** : On fournit au cuisinier les ingrédients et le four (injection). Il ne s'occupe que de cuisiner. On peut lui donner de la farine bio, de la farine standard ou de la farine de test sans toucher à sa recette.

L'injection de dépendances, c'est « fournir les ingrédients » au lieu de les faire produire par le composant lui-même.

## Ce que l'injection de dépendances n'est pas

- **Pas une fin en soi** : C'est un moyen pour obtenir du découplage et de la testabilité. On ne l'applique pas partout sans réfléchir.
- **Pas obligatoirement un framework** : On peut faire de l'injection de dépendances « à la main » en passant les dépendances dans le constructeur. Les conteneurs IoC (Spring, etc.) automatisent l'assemblage mais ne sont pas indispensables pour appliquer le principe.
- **Pas le Service Locator** : Le Service Locator fait que la classe va chercher elle-même ses dépendances dans un registre ; avec la DI, la classe les reçoit sans savoir d'où elles viennent.

## Quand l'injection de dépendances est utile

- Une classe a besoin d'un **service**, d'un **repository**, d'un **client** ou de tout autre collaborateur.
- Vous voulez **tester** cette classe en remplaçant les collaborateurs par des mocks.
- Vous voulez pouvoir **changer d'implémentation** (autre BDD, autre fournisseur d'email) sans modifier la classe.
- Vous visez une architecture **découplée** et alignée avec le **DIP**.

## Prochaines étapes

La suite de la documentation détaille :

- **[Types et mécanismes](./02-types-et-mecanismes.md)** : injection par constructeur, par setter, par interface ; rôle des conteneurs IoC
- **[Exemples concrets](./03-exemples-concrets.md)** : services métier, repositories, notifications, tests
- **[Mise en pratique](./04-mise-en-pratique.md)** : quand l'utiliser, pièges à éviter, intégration dans un projet

---

**Rappel** : L'injection de dépendances consiste à fournir les dépendances à un objet depuis l'extérieur, pour découpler, faciliter les tests et respecter le principe d'inversion des dépendances.
