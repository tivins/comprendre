# Dependency Inversion Principle (DIP)

## Définition

**Les modules de haut niveau ne devraient pas dépendre des modules de bas niveau. Les deux devraient dépendre d'abstractions.**

**Les abstractions ne devraient pas dépendre des détails. Les détails devraient dépendre des abstractions.**

Le principe d'inversion des dépendances (DIP) est le dernier principe SOLID. Il complète les autres principes en découplant les dépendances grâce aux abstractions.

## Comprendre DIP

### Modules de haut niveau vs bas niveau

- **Module de haut niveau** : Contient la logique métier, les règles de l'application
- **Module de bas niveau** : Contient les détails d'implémentation (BDD, fichiers, APIs externes)

### Le problème sans DIP

```php
// À éviter : Dépendance directe aux modules de bas niveau
class UserService {
    private $database;
    
    public function __construct() {
        $this->database = new MySQLDatabase(); // Couplage fort
    }
    
    public function getUser(int $id): ?User {
        return $this->database->query("SELECT * FROM users WHERE id = ?", [$id]);
    }
}
```

**Problèmes** :
- `UserService` (haut niveau) dépend directement de `MySQLDatabase` (bas niveau)
- Impossible de changer de base de données sans modifier `UserService`
- Difficile à tester (besoin d'une vraie base de données MySQL)
- Violation de l'OCP (modification nécessaire pour changer de BDD)

## Solution : Inverser les dépendances

```php
// Bon : Dépendance aux abstractions
interface UserRepository {
    public function findById(int $id): ?User;
    public function save(User $user): void;
}

class MySQLUserRepository implements UserRepository {
    private $connection;
    
    public function __construct(PDO $connection) {
        $this->connection = $connection;
    }
    
    public function findById(int $id): ?User {
        // Implémentation MySQL
    }
    
    public function save(User $user): void {
        // Implémentation MySQL
    }
}

class UserService {
    private $repository;
    
    // Dépend de l'abstraction, pas de l'implémentation
    public function __construct(UserRepository $repository) {
        $this->repository = $repository;
    }
    
    public function getUser(int $id): ?User {
        return $this->repository->findById($id);
    }
}

// Maintenant, UserService peut utiliser n'importe quelle implémentation
$mysqlRepo = new MySQLUserRepository($pdo);
$service = new UserService($mysqlRepo);

$postgresRepo = new PostgreSQLUserRepository($connection);
$service = new UserService($postgresRepo); // Même service, autre BDD
```

## Exemples détaillés

### Exemple 1 : Système de logging

```php
// À éviter : Dépendance directe à l'implémentation
class OrderService {
    private $fileLogger;
    
    public function __construct() {
        $this->fileLogger = new FileLogger('/var/log/app.log');
    }
    
    public function processOrder(Order $order): void {
        $this->fileLogger->log("Processing order: " . $order->getId());
        // Traitement de la commande
    }
}

// Bon : Dépendance à l'abstraction
interface Logger {
    public function log(string $message): void;
}

class FileLogger implements Logger {
    private string $filePath;
    
    public function __construct(string $filePath) {
        $this->filePath = $filePath;
    }
    
    public function log(string $message): void {
        file_put_contents($this->filePath, $message . "\n", FILE_APPEND);
    }
}

class DatabaseLogger implements Logger {
    private $connection;
    
    public function __construct(PDO $connection) {
        $this->connection = $connection;
    }
    
    public function log(string $message): void {
        $stmt = $this->connection->prepare("INSERT INTO logs (message) VALUES (?)");
        $stmt->execute([$message]);
    }
}

class CloudLogger implements Logger {
    private $apiKey;
    
    public function __construct(string $apiKey) {
        $this->apiKey = $apiKey;
    }
    
    public function log(string $message): void {
        // Envoi vers un service cloud (ex: Loggly, Datadog)
    }
}

class OrderService {
    private Logger $logger;
    
    // Dépend de l'abstraction Logger, pas d'une implémentation
    public function __construct(Logger $logger) {
        $this->logger = $logger;
    }
    
    public function processOrder(Order $order): void {
        $this->logger->log("Processing order: " . $order->getId());
        // Traitement de la commande
    }
}

// Utilisation flexible
$fileLogger = new FileLogger('/var/log/app.log');
$orderService = new OrderService($fileLogger);

$dbLogger = new DatabaseLogger($pdo);
$orderService = new OrderService($dbLogger);

$cloudLogger = new CloudLogger($apiKey);
$orderService = new OrderService($cloudLogger);
```

### Exemple 2 : Système de paiement

```php
// À éviter : Dépendance directe aux services de paiement
class PaymentService {
    private $stripe;
    
    public function __construct() {
        $this->stripe = new StripeClient(env('STRIPE_KEY'));
    }
    
    public function processPayment(float $amount, string $token): bool {
        return $this->stripe->charge($amount, $token);
    }
}

// Si on veut ajouter PayPal, il faut modifier PaymentService
class PaymentService {
    private $stripe;
    private $paypal;
    
    public function __construct() {
        $this->stripe = new StripeClient(env('STRIPE_KEY'));
        $this->paypal = new PayPalClient(env('PAYPAL_KEY'));
    }
    
    public function processPayment(float $amount, string $token, string $gateway): bool {
        if ($gateway === 'stripe') {
            return $this->stripe->charge($amount, $token);
        } elseif ($gateway === 'paypal') {
            return $this->paypal->charge($amount, $token);
        }
        // Violation de l'OCP : modification nécessaire pour chaque nouveau gateway
    }
}

// Bon : Dépendance à l'abstraction
interface PaymentGateway {
    public function charge(float $amount, string $token): bool;
    public function refund(string $transactionId, float $amount): bool;
}

class StripeGateway implements PaymentGateway {
    private $client;
    
    public function __construct(string $apiKey) {
        $this->client = new StripeClient($apiKey);
    }
    
    public function charge(float $amount, string $token): bool {
        return $this->client->charge($amount, $token);
    }
    
    public function refund(string $transactionId, float $amount): bool {
        return $this->client->refund($transactionId, $amount);
    }
}

class PayPalGateway implements PaymentGateway {
    private $client;
    
    public function __construct(string $apiKey) {
        $this->client = new PayPalClient($apiKey);
    }
    
    public function charge(float $amount, string $token): bool {
        return $this->client->charge($amount, $token);
    }
    
    public function refund(string $transactionId, float $amount): bool {
        return $this->client->refund($transactionId, $amount);
    }
}

class PaymentService {
    private PaymentGateway $gateway;
    
    // Dépend de l'abstraction PaymentGateway
    public function __construct(PaymentGateway $gateway) {
        $this->gateway = $gateway;
    }
    
    public function processPayment(float $amount, string $token): bool {
        return $this->gateway->charge($amount, $token);
    }
    
    public function processRefund(string $transactionId, float $amount): bool {
        return $this->gateway->refund($transactionId, $amount);
    }
}

// Extension : Ajouter un nouveau gateway sans modifier PaymentService
class SquareGateway implements PaymentGateway {
    public function charge(float $amount, string $token): bool {
        // Implémentation Square
    }
    
    public function refund(string $transactionId, float $amount): bool {
        // Implémentation Square
    }
}

// Utilisation
$stripeGateway = new StripeGateway($stripeKey);
$paymentService = new PaymentService($stripeGateway);

$paypalGateway = new PayPalGateway($paypalKey);
$paymentService = new PaymentService($paypalGateway);

$squareGateway = new SquareGateway($squareKey);
$paymentService = new PaymentService($squareGateway);
```

### Exemple 3 : Envoi de notifications

```php
// À éviter : Dépendance directe aux services de notification
class NotificationService {
    private $mailer;
    private $smsService;
    
    public function __construct() {
        $this->mailer = new PHPMailer();
        $this->smsService = new TwilioSMS();
    }
    
    public function sendEmail(string $to, string $subject, string $body): void {
        $this->mailer->send($to, $subject, $body);
    }
    
    public function sendSMS(string $to, string $message): void {
        $this->smsService->send($to, $message);
    }
}

// Bon : Dépendance aux abstractions
interface NotificationChannel {
    public function send(string $to, string $message): bool;
}

class EmailChannel implements NotificationChannel {
    private $mailer;
    
    public function __construct($mailer) {
        $this->mailer = $mailer;
    }
    
    public function send(string $to, string $message): bool {
        return $this->mailer->send($to, "Notification", $message);
    }
}

class SMSChannel implements NotificationChannel {
    private $smsService;
    
    public function __construct($smsService) {
        $this->smsService = $smsService;
    }
    
    public function send(string $to, string $message): bool {
        return $this->smsService->send($to, $message);
    }
}

class PushNotificationChannel implements NotificationChannel {
    private $pushService;
    
    public function __construct($pushService) {
        $this->pushService = $pushService;
    }
    
    public function send(string $to, string $message): bool {
        return $this->pushService->send($to, $message);
    }
}

class NotificationService {
    private NotificationChannel $channel;
    
    // Dépend de l'abstraction NotificationChannel
    public function __construct(NotificationChannel $channel) {
        $this->channel = $channel;
    }
    
    public function notify(string $recipient, string $message): bool {
        return $this->channel->send($recipient, $message);
    }
}

// Utilisation
$emailChannel = new EmailChannel($mailer);
$notificationService = new NotificationService($emailChannel);

$smsChannel = new SMSChannel($twilio);
$notificationService = new NotificationService($smsChannel);
```

### Exemple 4 : Cache

```php
// À éviter : Dépendance directe à Redis
class ProductService {
    private $redis;
    
    public function __construct() {
        $this->redis = new Redis();
        $this->redis->connect('127.0.0.1', 6379);
    }
    
    public function getProduct(int $id): ?Product {
        $cached = $this->redis->get("product:$id");
        if ($cached) {
            return unserialize($cached);
        }
        
        $product = $this->fetchFromDatabase($id);
        $this->redis->set("product:$id", serialize($product));
        return $product;
    }
}

// Bon : Dépendance à l'abstraction
interface Cache {
    public function get(string $key): mixed;
    public function set(string $key, mixed $value, int $ttl = 3600): bool;
    public function delete(string $key): bool;
}

class RedisCache implements Cache {
    private $redis;
    
    public function __construct(Redis $redis) {
        $this->redis = $redis;
    }
    
    public function get(string $key): mixed {
        $value = $this->redis->get($key);
        return $value ? unserialize($value) : null;
    }
    
    public function set(string $key, mixed $value, int $ttl = 3600): bool {
        return $this->redis->setex($key, $ttl, serialize($value));
    }
    
    public function delete(string $key): bool {
        return $this->redis->del($key) > 0;
    }
}

class MemcachedCache implements Cache {
    private $memcached;
    
    public function __construct(Memcached $memcached) {
        $this->memcached = $memcached;
    }
    
    public function get(string $key): mixed {
        return $this->memcached->get($key);
    }
    
    public function set(string $key, mixed $value, int $ttl = 3600): bool {
        return $this->memcached->set($key, $value, $ttl);
    }
    
    public function delete(string $key): bool {
        return $this->memcached->delete($key);
    }
}

class ArrayCache implements Cache {
    private array $cache = [];
    
    public function get(string $key): mixed {
        return $this->cache[$key] ?? null;
    }
    
    public function set(string $key, mixed $value, int $ttl = 3600): bool {
        $this->cache[$key] = $value;
        return true;
    }
    
    public function delete(string $key): bool {
        unset($this->cache[$key]);
        return true;
    }
}

class ProductService {
    private Cache $cache;
    private ProductRepository $repository;
    
    // Dépend des abstractions Cache et ProductRepository
    public function __construct(Cache $cache, ProductRepository $repository) {
        $this->cache = $cache;
        $this->repository = $repository;
    }
    
    public function getProduct(int $id): ?Product {
        $cached = $this->cache->get("product:$id");
        if ($cached !== null) {
            return $cached;
        }
        
        $product = $this->repository->findById($id);
        if ($product) {
            $this->cache->set("product:$id", $product);
        }
        
        return $product;
    }
}

// Utilisation flexible
$redisCache = new RedisCache($redis);
$productService = new ProductService($redisCache, $repository);

$memcachedCache = new MemcachedCache($memcached);
$productService = new ProductService($memcachedCache, $repository);

// Pour les tests
$arrayCache = new ArrayCache();
$productService = new ProductService($arrayCache, $mockRepository);
```

## Injection de dépendances

Le DIP va souvent de pair avec l'**injection de dépendances** (Dependency Injection).

### Types d'injection

#### 1. Injection par constructeur (recommandé)

```php
class UserService {
    public function __construct(
        private UserRepository $repository,
        private Logger $logger
    ) {}
}
```

#### 2. Injection par setter

```php
class UserService {
    private ?UserRepository $repository = null;
    
    public function setRepository(UserRepository $repository): void {
        $this->repository = $repository;
    }
}
```

#### 3. Injection par méthode

```php
class UserService {
    public function getUser(UserRepository $repository, int $id): ?User {
        return $repository->findById($id);
    }
}
```

### Conteneur d'injection de dépendances

```php
// Exemple simple de conteneur DI
class Container {
    private array $bindings = [];
    
    public function bind(string $abstract, callable $factory): void {
        $this->bindings[$abstract] = $factory;
    }
    
    public function resolve(string $abstract): mixed {
        if (isset($this->bindings[$abstract])) {
            return ($this->bindings[$abstract])($this);
        }
        
        throw new Exception("No binding for {$abstract}");
    }
}

// Configuration
$container = new Container();
$container->bind(UserRepository::class, function($container) {
    return new MySQLUserRepository($container->resolve(PDO::class));
});
$container->bind(PDO::class, function() {
    return new PDO('mysql:host=localhost;dbname=app', 'user', 'pass');
});
$container->bind(Logger::class, function() {
    return new FileLogger('/var/log/app.log');
});

// Utilisation
$userService = new UserService(
    $container->resolve(UserRepository::class),
    $container->resolve(Logger::class)
);
```

## Avantages du DIP

1. **Découplage** : Les modules de haut niveau ne dépendent pas des détails
2. **Testabilité** : Facile de mocker les dépendances pour les tests
3. **Flexibilité** : Facile de changer d'implémentation
4. **Réutilisabilité** : Les modules de haut niveau sont réutilisables
5. **Maintenabilité** : Modifications localisées aux implémentations

## Comment appliquer le DIP

1. **Identifier** les dépendances aux modules de bas niveau
2. **Créer** des interfaces pour ces dépendances
3. **Injecter** les dépendances plutôt que de les instancier
4. **Implémenter** les interfaces dans les modules de bas niveau
5. **Utiliser** l'injection de dépendances dans les modules de haut niveau

## DIP et les autres principes SOLID

- **SRP** : DIP aide à séparer les responsabilités en découplant les modules
- **OCP** : DIP facilite l'extension en permettant de changer d'implémentation
- **LSP** : DIP utilise LSP pour garantir la substituabilité
- **ISP** : DIP utilise ISP pour créer des abstractions appropriées

## Conclusion

Le principe d'inversion des dépendances complète les autres principes SOLID en découplant les modules grâce aux abstractions. En dépendant des abstractions plutôt que des implémentations concrètes, vous créez un code plus flexible, testable et maintenable. **Dépendre des abstractions, pas des détails**.
