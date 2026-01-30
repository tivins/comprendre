# Exemples concrets d'injection de dépendances

Ce chapitre présente des exemples concrets en pseudo-code Java : service métier avec repository et notification, tests unitaires avec mocks, et composition root sans framework.

## Exemple 1 : Service métier, repository et notification

### Contexte

Une application de gestion de commandes : un `OrderService` enregistre une commande, met à jour le stock et envoie une notification au client. On souhaite pouvoir tester le service sans vraie base de données ni envoi d'email, et pouvoir changer d'implémentation (BDD, fournisseur d'email) sans toucher au service.

### Abstractions (interfaces)

```java
interface OrderRepository {
    void save(Order order);
    Order findById(String id);
}

interface StockService {
    void reserve(String productId, int quantity);
    void release(String productId, int quantity);
}

interface NotificationService {
    void sendOrderConfirmation(Order order);
}
```

### Implémentations concrètes

```java
class JdbcOrderRepository implements OrderRepository {
    private final DataSource dataSource;

    public JdbcOrderRepository(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public void save(Order order) {
        // INSERT dans la BDD via JDBC
    }

    public Order findById(String id) {
        // SELECT depuis la BDD
        return null; // simplifié
    }
}

class SmtpNotificationService implements NotificationService {
    private final String smtpHost;
    private final int smtpPort;

    public SmtpNotificationService(String smtpHost, int smtpPort) {
        this.smtpHost = smtpHost;
        this.smtpPort = smtpPort;
    }

    public void sendOrderConfirmation(Order order) {
        // Envoi email via SMTP
    }
}
```

### Service métier avec injection par constructeur

```java
class OrderService {
    private final OrderRepository orderRepository;
    private final StockService stockService;
    private final NotificationService notificationService;

    public OrderService(OrderRepository orderRepository,
                        StockService stockService,
                        NotificationService notificationService) {
        this.orderRepository = orderRepository;
        this.stockService = stockService;
        this.notificationService = notificationService;
    }

    public void placeOrder(Order order) {
        for (OrderLine line : order.getLines()) {
            stockService.reserve(line.getProductId(), line.getQuantity());
        }
        orderRepository.save(order);
        notificationService.sendOrderConfirmation(order);
    }
}
```

`OrderService` ne connaît que les interfaces. Il ne crée aucun objet concret ; toutes ses dépendances sont injectées.

### Assemblage en production (composition root)

```java
class Application {
    public static void main(String[] args) {
        DataSource dataSource = createDataSource();
        OrderRepository orderRepo = new JdbcOrderRepository(dataSource);
        StockService stockService = new DefaultStockService(dataSource);
        NotificationService notificationService = new SmtpNotificationService("smtp.example.com", 587);

        OrderService orderService = new OrderService(orderRepo, stockService, notificationService);

        // Utilisation
        Order order = buildOrderFromRequest(request);
        orderService.placeOrder(order);
    }
}
```

### Assemblage en test avec mocks

```java
class OrderServiceTest {
    @Test
    void placeOrder_reservesStock_savesOrder_sendsNotification() {
        OrderRepository mockRepo = mock(OrderRepository.class);
        StockService mockStock = mock(StockService.class);
        NotificationService mockNotif = mock(NotificationService.class);

        OrderService service = new OrderService(mockRepo, mockStock, mockNotif);
        Order order = new Order("order-1", List.of(new OrderLine("prod-A", 2)));

        service.placeOrder(order);

        verify(mockStock).reserve("prod-A", 2);
        verify(mockRepo).save(order);
        verify(mockNotif).sendOrderConfirmation(order);
    }
}
```

Grâce à l'injection, on remplace chaque dépendance par un mock ; le comportement de `OrderService` peut être vérifié sans base de données ni SMTP.

---

## Exemple 2 : Utilisateur et authentification

### Contexte

Un service d'authentification qui vérifie le mot de passe via un `PasswordEncoder` et enregistre les événements de connexion via un `AuditLog`. On veut pouvoir tester avec un encodeur factice et un audit en mémoire.

### Code

```java
interface PasswordEncoder {
    String encode(String rawPassword);
    boolean matches(String rawPassword, String encodedPassword);
}

interface AuditLog {
    void log(String event, String userId);
}

class AuthService {
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    private final AuditLog auditLog;

    public AuthService(UserRepository userRepository,
                       PasswordEncoder passwordEncoder,
                       AuditLog auditLog) {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
        this.auditLog = auditLog;
    }

    public boolean login(String username, String password) {
        User user = userRepository.findByUsername(username);
        if (user == null) {
            return false;
        }
        boolean ok = passwordEncoder.matches(password, user.getEncodedPassword());
        if (ok) {
            auditLog.log("LOGIN_SUCCESS", user.getId());
        } else {
            auditLog.log("LOGIN_FAILURE", user.getId());
        }
        return ok;
    }
}
```

En production : `BCryptPasswordEncoder`, `JdbcUserRepository`, `DatabaseAuditLog`. En test : encodeur qui compare en clair, repository en mémoire, audit en mémoire ou mock. L'injection permet d'alterner sans modifier `AuthService`.

---

## Exemple 3 : Dépendance optionnelle (setter)

### Contexte

Un `ReportGenerator` qui génère un rapport. Un `Logger` est optionnel : s'il est fourni, on log ; sinon on ne fait rien. On utilise l'injection par setter pour cette dépendance optionnelle.

```java
interface Logger {
    void info(String message);
}

class ReportGenerator {
    private final DataSource dataSource;
    private Logger logger; // optionnel, peut rester null

    public ReportGenerator(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    public void setLogger(Logger logger) {
        this.logger = logger;
    }

    public Report generate(String reportId) {
        if (logger != null) {
            logger.info("Generating report " + reportId);
        }
        // ... génération du rapport
        return report;
    }
}

// Production : avec logger
ReportGenerator generator = new ReportGenerator(dataSource);
generator.setLogger(new Slf4jLogger());

// Test : sans logger (pas besoin d'appeler setLogger)
ReportGenerator generator = new ReportGenerator(mockDataSource);
```

Ici, le constructeur ne reçoit que la dépendance obligatoire (`DataSource`) ; le logger est optionnel et injecté par setter.

---

## Exemple 4 : Plusieurs implémentations (stratégie)

### Contexte

Un service de paiement qui peut accepter plusieurs moyens de paiement (carte, PayPal, virement). Chaque moyen est une implémentation d'une interface `PaymentProcessor`. Le service reçoit la liste des processeurs par constructeur.

```java
interface PaymentProcessor {
    boolean supports(String method);
    PaymentResult process(PaymentRequest request);
}

class PaymentService {
    private final List<PaymentProcessor> processors;

    public PaymentService(List<PaymentProcessor> processors) {
        this.processors = processors;
    }

    public PaymentResult pay(PaymentRequest request) {
        for (PaymentProcessor processor : processors) {
            if (processor.supports(request.getMethod())) {
                return processor.process(request);
            }
        }
        throw new UnsupportedPaymentMethodException(request.getMethod());
    }
}

// Assemblage
List<PaymentProcessor> processors = List.of(
    new CardPaymentProcessor(apiKey),
    new PayPalPaymentProcessor(clientId, secret),
    new BankTransferProcessor(config)
);
PaymentService paymentService = new PaymentService(processors);
```

L'injection permet d'ajouter ou retirer un moyen de paiement en ne modifiant que la composition root, pas `PaymentService`.

---

## Exemple 5 : Composition root sans framework

### Contexte

Une petite application sans framework IoC : tout l'assemblage est fait manuellement dans une classe `CompositionRoot` ou dans le `main`.

```java
class CompositionRoot {
    private final Config config;
    private final DataSource dataSource;
    private final OrderRepository orderRepository;
    private final NotificationService notificationService;
    private final OrderService orderService;

    public CompositionRoot(Config config) {
        this.config = config;
        this.dataSource = createDataSource(config.getJdbcUrl());
        this.orderRepository = new JdbcOrderRepository(dataSource);
        this.notificationService = new SmtpNotificationService(
            config.getSmtpHost(),
            config.getSmtpPort()
        );
        this.orderService = new OrderService(orderRepository, notificationService);
    }

    public OrderService getOrderService() {
        return orderService;
    }

    private DataSource createDataSource(String jdbcUrl) {
        // Création du pool de connexions
        return null; // simplifié
    }
}

// Point d'entrée
class Main {
    public static void main(String[] args) {
        Config config = Config.fromEnv();
        CompositionRoot root = new CompositionRoot(config);
        OrderService orderService = root.getOrderService();
        // Lancer le serveur HTTP, la CLI, etc. en utilisant orderService
    }
}
```

Toutes les dépendances sont créées à un seul endroit ; le reste de l'application ne fait que des `getXxx()` ou reçoit les services déjà construits. C'est de l'injection de dépendances « manuelle », sans conteneur IoC.

---

## Récapitulatif des exemples

| Exemple              | Type d'injection      | Intérêt principal                          |
|----------------------|------------------------|--------------------------------------------|
| OrderService         | Constructeur           | Découplage, testabilité avec mocks         |
| AuthService          | Constructeur           | Même idée ; plusieurs dépendances         |
| ReportGenerator      | Constructeur + setter  | Dépendance optionnelle (logger)            |
| PaymentService       | Constructeur (liste)   | Plusieurs implémentations (stratégies)    |
| Composition root     | Manuelle               | DI sans framework, assemblage centralisé  |

En pratique : **privilégier l'injection par constructeur** pour les dépendances obligatoires, et **centraliser l'assemblage** dans une composition root ou un conteneur IoC. La suite propose un guide de **[mise en pratique](./04-mise-en-pratique.md)** (quand utiliser la DI, pièges à éviter).
