# Types et mécanismes d'injection de dépendances

Il existe plusieurs façons d'injecter des dépendances dans un objet. Ce chapitre présente les trois formes classiques (constructeur, setter, interface) et le rôle des conteneurs IoC.

## Les trois formes d'injection

Martin Fowler distingue trois types d'injection de dépendances :

1. **Injection par constructeur** (Constructor Injection)
2. **Injection par setter** (Setter Injection)
3. **Injection par interface** (Interface Injection)

En pratique, l'injection par constructeur est la plus utilisée et la plus recommandée pour la majorité des cas.

---

## 1. Injection par constructeur (Constructor Injection)

### Principe

Les dépendances sont passées via le constructeur de la classe. L'objet est construit déjà « complet » : une fois créé, toutes ses dépendances sont en place.

### Exemple en pseudo-code Java

```java
interface NotificationService {
    void notify(User user, String message);
}

class UserService {
    private final UserRepository userRepository;
    private final NotificationService notificationService;

    public UserService(UserRepository userRepository, NotificationService notificationService) {
        this.userRepository = userRepository;
        this.notificationService = notificationService;
    }

    public void register(User user) {
        userRepository.save(user);
        notificationService.notify(user, "Bienvenue !");
    }
}

// Assemblage
UserRepository repo = new InMemoryUserRepository();
NotificationService notif = new EmailNotificationService();
UserService userService = new UserService(repo, notif);
```

### Avantages

- **Dépendances immuables** : On peut stocker les champs en `final`, ce qui garantit qu'elles ne changent pas après construction.
- **Objet toujours dans un état valide** : Dès la création, l'objet a tout ce qu'il faut pour fonctionner ; pas d'état « à moitié initialisé ».
- **Explicite** : La signature du constructeur montre clairement toutes les dépendances requises.
- **Facile à tester** : On construit l'objet une fois avec des mocks, sans avoir à appeler des setters.
- **Compatible avec l'immutabilité** : Idéal pour des objets qui ne doivent pas être modifiés après création.

### Inconvénients

- Si une classe a **beaucoup** de dépendances (par exemple plus de 4 ou 5), le constructeur devient lourd. Cela peut être un signal qu'il faut repenser la responsabilité de la classe (principe SRP).

### Quand l'utiliser

- **Par défaut** : C'est le choix recommandé pour la plupart des services, repositories, handlers, etc.
- Lorsque les dépendances sont **obligatoires** pour que l'objet soit utilisable.

---

## 2. Injection par setter (Setter Injection)

### Principe

Les dépendances sont fournies via des méthodes setter après la construction de l'objet. L'objet peut être créé avec un constructeur vide (ou avec des paramètres optionnels), puis configuré via des setters.

### Exemple en pseudo-code Java

```java
class ReportGenerator {
    private PdfExporter pdfExporter;
    private EmailSender emailSender;

    public ReportGenerator() {
        // Objet créé sans dépendances
    }

    public void setPdfExporter(PdfExporter pdfExporter) {
        this.pdfExporter = pdfExporter;
    }

    public void setEmailSender(EmailSender emailSender) {
        this.emailSender = emailSender;
    }

    public void generateAndSend(Report report) {
        byte[] pdf = pdfExporter.export(report);
        emailSender.send(report.getRecipient(), "Rapport", pdf);
    }
}

// Assemblage
ReportGenerator generator = new ReportGenerator();
generator.setPdfExporter(new DefaultPdfExporter());
generator.setEmailSender(new SmtpEmailSender());
```

### Avantages

- Permet de **modifier** une dépendance après la création (par exemple pour reconfigurer ou remplacer un service).
- Utile lorsque certaines dépendances sont **optionnelles** ou ont une **valeur par défaut**.
- Certains frameworks historiques (ex. Spring XML) utilisaient beaucoup les setters pour la configuration.

### Inconvénients

- L'objet peut être utilisé **avant** que tous les setters aient été appelés, ce qui mène à des erreurs à l'exécution (NullPointerException, comportement incorrect).
- Les dépendances ne sont pas visibles dans la signature du constructeur ; il faut lire la classe pour savoir quels setters sont requis.
- Moins adapté à l'immutabilité et aux champs `final`.

### Quand l'utiliser

- Dépendances **optionnelles** ou avec valeur par défaut.
- Cas où le framework ou l'API impose des setters (par exemple beans Java avec convention setXxx).
- À utiliser avec prudence ; privilégier l'injection par constructeur dès que les dépendances sont requises.

---

## 3. Injection par interface (Interface Injection)

### Principe

La classe dépend d'une **interface** qui expose une méthode permettant d'injecter la dépendance. L'interface définit un « contrat d'injection » : par exemple `setDependency(Dependency d)`.

### Exemple en pseudo-code Java

```java
interface EmailServiceInjector {
    void setEmailService(EmailService emailService);
}

interface OrderRepositoryInjector {
    void setOrderRepository(OrderRepository orderRepository);
}

class OrderService implements EmailServiceInjector, OrderRepositoryInjector {
    private EmailService emailService;
    private OrderRepository orderRepository;

    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }

    public void setOrderRepository(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }

    public void placeOrder(Order order) {
        orderRepository.save(order);
        emailService.send(order.getCustomerEmail(), "Confirmation", "...");
    }
}
```

### Avantages

- Permet à un conteneur ou à un assembleur de savoir **quelle dépendance** injecter en se basant sur le type d'interface (ex. « tout ce qui implémente EmailServiceInjector reçoit l'EmailService »).

### Inconvénients

- **Peu utilisé** en pratique ; ajoute des interfaces spécifiques à la DI sans apporter beaucoup plus que l'injection par constructeur ou setter.
- Prolifération d'interfaces (« injectors ») si chaque dépendance a la sienne.

### Quand l'utiliser

- Cas particuliers où un framework ou un pattern exige un contrat d'injection explicite. En général, l'injection par constructeur ou par setter suffit.

---

## Comparaison rapide

| Critère                    | Constructeur | Setter   | Interface   |
|---------------------------|-------------|----------|-------------|
| Dépendances obligatoires  | Oui, naturel | À gérer  | À gérer     |
| Immuabilité               | Oui (final) | Non      | Non         |
| Lisibilité des dépendances| Très bonne  | Moyenne  | Moyenne     |
| Usage courant             | Recommandé  | Optionnel| Rare        |

**Recommandation** : privilégier l'**injection par constructeur** partout où les dépendances sont requises ; utiliser le setter pour des dépendances optionnelles ou imposées par le contexte (framework, legacy).

---

## Conteneurs IoC (Inversion of Control)

### Qu'est-ce qu'un conteneur IoC ?

Un **conteneur IoC** (ou conteneur d'injection de dépendances) est un composant qui :

1. **Enregistre** les abstractions (interfaces) et leurs implémentations (classes concrètes).
2. **Résout** les dépendances : lorsqu'on demande une instance d'un type (ex. `OrderService`), le conteneur crée les dépendances nécessaires (ex. `OrderRepository`, `EmailService`) et les injecte.
3. **Gère le cycle de vie** des objets (singleton, une instance par requête, etc.).

En résumé : au lieu d'écrire vous-même tout l'assemblage (`new A(new B(), new C())`), vous déclarez « comment construire A, B, C » et le conteneur construit l'arbre d'objets à votre place.

### Inversion of Control (IoC)

L'**inversion de contrôle** signifie que le flux de contrôle est inversé : au lieu que votre code appelle un framework (ou crée ses dépendances), c'est le **framework/conteneur** qui appelle votre code et lui fournit ce dont il a besoin. L'injection de dépendances est une forme concrète d'IoC : le conteneur « contrôle » la création et l'injection des dépendances.

### Exemple conceptuel (style Spring)

En pseudo-code, l'idée d'un conteneur ressemble à ceci :

```java
// Déclaration (configuration ou annotations)
// "OrderService dépend de OrderRepository et EmailService"
// "OrderRepository → JdbcOrderRepository (singleton)"
// "EmailService → SmtpEmailService (singleton)"

// Utilisation : on ne fait plus new OrderService(...) soi-même
OrderService orderService = container.get(OrderService.class);
orderService.placeOrder(order);
```

Le conteneur :

- Voit que `OrderService` a un constructeur avec `OrderRepository` et `EmailService`.
- Crée ou récupère des instances de `OrderRepository` et `EmailService` (selon la config).
- Appelle le constructeur de `OrderService` avec ces instances.

### Avantages d'un conteneur

- **Moins de code d'assemblage** : plus besoin d'écrire manuellement toutes les instanciations à la racine de l'application.
- **Gestion du scope** : singleton, request-scoped, etc., sans dupliquer la logique.
- **Configuration centralisée** : on peut changer d'implémentation (ex. passer de JDBC à JPA) en modifiant la configuration plutôt que le code métier.

### Sans conteneur : composition root

On peut tout à fait faire de l'injection de dépendances **sans** framework : il suffit de centraliser la création des objets dans un endroit (souvent appelé **composition root** ou **assemblage**) et de passer les dépendances par constructeur. C'est une approche valide et parfois préférable pour des projets simples ou pour bien comprendre le principe avant d'utiliser Spring ou un équivalent.

```java
class Application {
    public static void main(String[] args) {
        // Composition root : tout l'assemblage est ici
        OrderRepository orderRepo = new JdbcOrderRepository(dataSource);
        EmailService emailService = new SmtpEmailService(config);
        OrderService orderService = new OrderService(orderRepo, emailService);

        // Ensuite on utilise orderService, etc.
    }
}
```

---

## Récapitulatif

- **Injection par constructeur** : dépendances passées au constructeur ; à privilégier pour des dépendances obligatoires et pour garder des objets immuables.
- **Injection par setter** : dépendances fournies via des setters ; utile pour des dépendances optionnelles ou imposées par le contexte.
- **Injection par interface** : contrat d'injection via une interface ; peu utilisée en pratique.
- **Conteneur IoC** : automatise l'assemblage et la résolution des dépendances ; optionnel mais pratique dans les grosses applications.

La suite propose des **[exemples concrets](./03-exemples-concrets.md)** en pseudo-code Java (services, repositories, tests) et un guide de **[mise en pratique](./04-mise-en-pratique.md)**.
