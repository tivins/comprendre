# Exemples Concrets en PHP

Ce document présente des exemples complets et réalistes d'application des principes SOLID dans des projets PHP concrets. Chaque exemple montre comment plusieurs principes SOLID travaillent ensemble pour créer un code de qualité.

## Exemple 1 : Système de gestion de commandes E-Commerce

### Contexte

Nous construisons un système e-commerce avec :
- Gestion des commandes
- Calcul des prix avec différentes remises
- Traitement des paiements multiples
- Envoi de notifications
- Génération de factures

### Analyse SOLID

- **SRP** : Séparation des responsabilités (commandes, paiements, notifications, factures)
- **OCP** : Extension pour nouveaux types de remises et méthodes de paiement
- **LSP** : Toutes les méthodes de paiement sont substituables
- **ISP** : Interfaces spécifiques pour chaque capacité
- **DIP** : Dépendances injectées via interfaces

### Implémentation

```php
<?php

// ========== SRP : Séparation des responsabilités ==========

// Entité métier : Commande
class Order {
    private int $id;
    private array $items;
    private float $total;
    private string $status;
    
    public function __construct(int $id, array $items) {
        $this->id = $id;
        $this->items = $items;
        $this->status = 'pending';
    }
    
    public function getId(): int {
        return $this->id;
    }
    
    public function getItems(): array {
        return $this->items;
    }
    
    public function getTotal(): float {
        return $this->total;
    }
    
    public function setTotal(float $total): void {
        $this->total = $total;
    }
    
    public function getStatus(): string {
        return $this->status;
    }
    
    public function setStatus(string $status): void {
        $this->status = $status;
    }
}

// ========== OCP + ISP : Interfaces pour les remises ==========

interface DiscountStrategy {
    public function calculate(float $price): float;
}

class NoDiscount implements DiscountStrategy {
    public function calculate(float $price): float {
        return $price;
    }
}

class PercentageDiscount implements DiscountStrategy {
    public function __construct(private float $percentage) {}
    
    public function calculate(float $price): float {
        return $price * (1 - $this->percentage / 100);
    }
}

class FixedDiscount implements DiscountStrategy {
    public function __construct(private float $amount) {}
    
    public function calculate(float $price): float {
        return max(0, $price - $this->amount);
    }
}

class SeasonalDiscount implements DiscountStrategy {
    public function calculate(float $price): float {
        $month = (int)date('m');
        // Réduction de 20% en décembre
        return $month === 12 ? $price * 0.8 : $price;
    }
}

// ========== SRP : Service de calcul de prix ==========

class PriceCalculator {
    public function __construct(private DiscountStrategy $discount) {}
    
    public function calculateTotal(Order $order): float {
        $total = 0;
        foreach ($order->getItems() as $item) {
            $itemPrice = $item['price'] * $item['quantity'];
            $total += $this->discount->calculate($itemPrice);
        }
        return $total;
    }
}

// ========== ISP + DIP : Interfaces pour les paiements ==========

interface PaymentGateway {
    public function charge(float $amount, array $paymentData): PaymentResult;
}

interface RefundableGateway {
    public function refund(string $transactionId, float $amount): bool;
}

class PaymentResult {
    public function __construct(
        private bool $success,
        private string $transactionId,
        private string $message = ''
    ) {}
    
    public function isSuccess(): bool {
        return $this->success;
    }
    
    public function getTransactionId(): string {
        return $this->transactionId;
    }
    
    public function getMessage(): string {
        return $this->message;
    }
}

// ========== OCP : Implémentations de paiement ==========

class StripeGateway implements PaymentGateway, RefundableGateway {
    public function charge(float $amount, array $paymentData): PaymentResult {
        // Simulation Stripe
        $transactionId = 'stripe_' . uniqid();
        return new PaymentResult(true, $transactionId, 'Stripe payment successful');
    }
    
    public function refund(string $transactionId, float $amount): bool {
        // Simulation remboursement Stripe
        return true;
    }
}

class PayPalGateway implements PaymentGateway, RefundableGateway {
    public function charge(float $amount, array $paymentData): PaymentResult {
        // Simulation PayPal
        $transactionId = 'paypal_' . uniqid();
        return new PaymentResult(true, $transactionId, 'PayPal payment successful');
    }
    
    public function refund(string $transactionId, float $amount): bool {
        // Simulation remboursement PayPal
        return true;
    }
}

class CashOnDeliveryGateway implements PaymentGateway {
    public function charge(float $amount, array $paymentData): PaymentResult {
        // Pas de paiement immédiat pour le paiement à la livraison
        $transactionId = 'cod_' . uniqid();
        return new PaymentResult(true, $transactionId, 'Order will be paid on delivery');
    }
}

// ========== SRP : Service de paiement ==========

class PaymentService {
    public function __construct(private PaymentGateway $gateway) {}
    
    public function processPayment(Order $order, array $paymentData): PaymentResult {
        return $this->gateway->charge($order->getTotal(), $paymentData);
    }
}

// ========== ISP : Interfaces pour les notifications ==========

interface EmailNotifiable {
    public function sendEmail(string $to, string $subject, string $body): bool;
}

interface SMSNotifiable {
    public function sendSMS(string $to, string $message): bool;
}

interface PushNotifiable {
    public function sendPush(string $userId, string $message): bool;
}

// ========== SRP : Services de notification ==========

class EmailService implements EmailNotifiable {
    public function sendEmail(string $to, string $subject, string $body): bool {
        // Simulation envoi email
        echo "Email sent to {$to}: {$subject}\n";
        return true;
    }
}

class SMSService implements SMSNotifiable {
    public function sendSMS(string $to, string $message): bool {
        // Simulation envoi SMS
        echo "SMS sent to {$to}: {$message}\n";
        return true;
    }
}

class PushNotificationService implements PushNotifiable {
    public function sendPush(string $userId, string $message): bool {
        // Simulation notification push
        echo "Push sent to user {$userId}: {$message}\n";
        return true;
    }
}

// ========== SRP : Service de notification de commande ==========

class OrderNotificationService {
    public function __construct(
        private EmailNotifiable $emailService,
        private ?SMSNotifiable $smsService = null,
        private ?PushNotifiable $pushService = null
    ) {}
    
    public function notifyOrderCreated(Order $order, string $customerEmail): void {
        $subject = "Order #{$order->getId()} Confirmed";
        $body = "Your order has been confirmed. Total: $" . $order->getTotal();
        
        $this->emailService->sendEmail($customerEmail, $subject, $body);
        
        if ($this->smsService) {
            $this->smsService->sendSMS($customerEmail, "Order #{$order->getId()} confirmed");
        }
        
        if ($this->pushService) {
            $this->pushService->sendPush($customerEmail, "Order confirmed");
        }
    }
}

// ========== ISP : Interfaces pour la génération de factures ==========

interface InvoiceGenerator {
    public function generate(Order $order): string; // Retourne le chemin du fichier
}

interface PDFGenerator {
    public function generatePDF(string $content): string;
}

interface HTMLGenerator {
    public function generateHTML(string $content): string;
}

// ========== SRP : Service de génération de factures ==========

class PDFInvoiceGenerator implements InvoiceGenerator, PDFGenerator {
    public function generate(Order $order): string {
        $content = $this->buildInvoiceContent($order);
        return $this->generatePDF($content);
    }
    
    private function buildInvoiceContent(Order $order): string {
        $content = "Invoice #{$order->getId()}\n";
        $content .= "Items:\n";
        foreach ($order->getItems() as $item) {
            $content .= "- {$item['name']}: $" . ($item['price'] * $item['quantity']) . "\n";
        }
        $content .= "Total: $" . $order->getTotal() . "\n";
        return $content;
    }
    
    public function generatePDF(string $content): string {
        // Simulation génération PDF
        $filename = 'invoice_' . uniqid() . '.pdf';
        file_put_contents($filename, $content);
        return $filename;
    }
}

// ========== DIP : Repository abstrait ==========

interface OrderRepository {
    public function save(Order $order): void;
    public function findById(int $id): ?Order;
}

class MySQLOrderRepository implements OrderRepository {
    public function __construct(private PDO $connection) {}
    
    public function save(Order $order): void {
        $stmt = $this->connection->prepare(
            "INSERT INTO orders (id, total, status) VALUES (?, ?, ?)"
        );
        $stmt->execute([$order->getId(), $order->getTotal(), $order->getStatus()]);
    }
    
    public function findById(int $id): ?Order {
        $stmt = $this->connection->prepare("SELECT * FROM orders WHERE id = ?");
        $stmt->execute([$id]);
        $data = $stmt->fetch(PDO::FETCH_ASSOC);
        
        if (!$data) {
            return null;
        }
        
        // Reconstruction de l'objet Order
        // ...
        return new Order($data['id'], []);
    }
}

// ========== SRP : Service principal de commande ==========

class OrderService {
    public function __construct(
        private OrderRepository $repository,
        private PriceCalculator $priceCalculator,
        private PaymentService $paymentService,
        private OrderNotificationService $notificationService,
        private InvoiceGenerator $invoiceGenerator
    ) {}
    
    public function processOrder(Order $order, array $paymentData, string $customerEmail): bool {
        // 1. Calculer le prix total
        $total = $this->priceCalculator->calculateTotal($order);
        $order->setTotal($total);
        
        // 2. Traiter le paiement
        $paymentResult = $this->paymentService->processPayment($order, $paymentData);
        
        if (!$paymentResult->isSuccess()) {
            return false;
        }
        
        // 3. Sauvegarder la commande
        $order->setStatus('paid');
        $this->repository->save($order);
        
        // 4. Envoyer les notifications
        $this->notificationService->notifyOrderCreated($order, $customerEmail);
        
        // 5. Générer la facture
        $invoicePath = $this->invoiceGenerator->generate($order);
        
        return true;
    }
}

// ========== Utilisation ==========

// Configuration
$pdo = new PDO('mysql:host=localhost;dbname=ecommerce', 'user', 'pass');
$orderRepository = new MySQLOrderRepository($pdo);

$discount = new PercentageDiscount(10); // 10% de réduction
$priceCalculator = new PriceCalculator($discount);

$paymentGateway = new StripeGateway();
$paymentService = new PaymentService($paymentGateway);

$emailService = new EmailService();
$smsService = new SMSService();
$notificationService = new OrderNotificationService($emailService, $smsService);

$invoiceGenerator = new PDFInvoiceGenerator();

// Création du service principal
$orderService = new OrderService(
    $orderRepository,
    $priceCalculator,
    $paymentService,
    $notificationService,
    $invoiceGenerator
);

// Traitement d'une commande
$order = new Order(1, [
    ['name' => 'Product A', 'price' => 100, 'quantity' => 2],
    ['name' => 'Product B', 'price' => 50, 'quantity' => 1]
]);

$orderService->processOrder($order, ['token' => 'stripe_token'], 'customer@example.com');
```

### Points SOLID respectés

1. **SRP** : Chaque classe a une seule responsabilité
2. **OCP** : Nouveaux types de remises et paiements sans modifier le code existant
3. **LSP** : Toutes les implémentations sont substituables
4. **ISP** : Interfaces spécifiques (EmailNotifiable, SMSNotifiable, etc.)
5. **DIP** : Toutes les dépendances sont injectées via interfaces

## Exemple 2 : Système de gestion d'utilisateurs avec authentification

### Contexte

Système d'authentification avec :
- Inscription et connexion
- Validation des données
- Hachage des mots de passe
- Gestion des sessions
- Envoi d'emails de confirmation

### Implémentation

```php
<?php

// ========== SRP : Entité utilisateur ==========

class User {
    private int $id;
    private string $email;
    private string $passwordHash;
    private bool $isVerified;
    
    public function __construct(string $email, string $passwordHash) {
        $this->email = $email;
        $this->passwordHash = $passwordHash;
        $this->isVerified = false;
    }
    
    public function getId(): int {
        return $this->id;
    }
    
    public function getEmail(): string {
        return $this->email;
    }
    
    public function getPasswordHash(): string {
        return $this->passwordHash;
    }
    
    public function isVerified(): bool {
        return $this->isVerified;
    }
    
    public function verify(): void {
        $this->isVerified = true;
    }
}

// ========== ISP : Interfaces de validation ==========

interface Validator {
    public function validate($value): ValidationResult;
}

class ValidationResult {
    public function __construct(
        private bool $isValid,
        private string $errorMessage = ''
    ) {}
    
    public function isValid(): bool {
        return $this->isValid;
    }
    
    public function getErrorMessage(): string {
        return $this->errorMessage;
    }
}

// ========== OCP : Règles de validation extensibles ==========

class RequiredRule implements Validator {
    public function validate($value): ValidationResult {
        $isValid = !empty($value);
        return new ValidationResult($isValid, $isValid ? '' : 'Field is required');
    }
}

class EmailRule implements Validator {
    public function validate($value): ValidationResult {
        $isValid = filter_var($value, FILTER_VALIDATE_EMAIL) !== false;
        return new ValidationResult($isValid, $isValid ? '' : 'Invalid email format');
    }
}

class MinLengthRule implements Validator {
    public function __construct(private int $minLength) {}
    
    public function validate($value): ValidationResult {
        $isValid = strlen($value) >= $this->minLength;
        return new ValidationResult(
            $isValid,
            $isValid ? '' : "Must be at least {$this->minLength} characters"
        );
    }
}

class PasswordStrengthRule implements Validator {
    public function validate($value): ValidationResult {
        $hasUpper = preg_match('/[A-Z]/', $value);
        $hasLower = preg_match('/[a-z]/', $value);
        $hasNumber = preg_match('/[0-9]/', $value);
        
        $isValid = $hasUpper && $hasLower && $hasNumber && strlen($value) >= 8;
        
        return new ValidationResult(
            $isValid,
            $isValid ? '' : 'Password must contain uppercase, lowercase, number and be at least 8 characters'
        );
    }
}

// ========== SRP : Service de validation ==========

class ValidationService {
    private array $rules = [];
    
    public function addRule(string $field, Validator $rule): void {
        $this->rules[$field][] = $rule;
    }
    
    public function validate(array $data): array {
        $errors = [];
        
        foreach ($this->rules as $field => $fieldRules) {
            $value = $data[$field] ?? null;
            
            foreach ($fieldRules as $rule) {
                $result = $rule->validate($value);
                if (!$result->isValid()) {
                    $errors[$field][] = $result->getErrorMessage();
                }
            }
        }
        
        return $errors;
    }
}

// ========== DIP : Interface pour le hachage ==========

interface PasswordHasher {
    public function hash(string $password): string;
    public function verify(string $password, string $hash): bool;
}

class BcryptPasswordHasher implements PasswordHasher {
    public function hash(string $password): string {
        return password_hash($password, PASSWORD_BCRYPT);
    }
    
    public function verify(string $password, string $hash): bool {
        return password_verify($password, $hash);
    }
}

class Argon2PasswordHasher implements PasswordHasher {
    public function hash(string $password): string {
        return password_hash($password, PASSWORD_ARGON2ID);
    }
    
    public function verify(string $password, string $hash): bool {
        return password_verify($password, $hash);
    }
}

// ========== DIP : Interface pour le repository ==========

interface UserRepository {
    public function save(User $user): void;
    public function findByEmail(string $email): ?User;
    public function findById(int $id): ?User;
}

class MySQLUserRepository implements UserRepository {
    public function __construct(private PDO $connection) {}
    
    public function save(User $user): void {
        $stmt = $this->connection->prepare(
            "INSERT INTO users (email, password_hash, is_verified) VALUES (?, ?, ?)"
        );
        $stmt->execute([
            $user->getEmail(),
            $user->getPasswordHash(),
            $user->isVerified() ? 1 : 0
        ]);
    }
    
    public function findByEmail(string $email): ?User {
        // Implémentation...
        return null;
    }
    
    public function findById(int $id): ?User {
        // Implémentation...
        return null;
    }
}

// ========== ISP : Interface pour l'envoi d'emails ==========

interface EmailSender {
    public function send(string $to, string $subject, string $body): bool;
}

class SMTPEmailSender implements EmailSender {
    public function send(string $to, string $subject, string $body): bool {
        // Implémentation SMTP
        return mail($to, $subject, $body);
    }
}

class SendGridEmailSender implements EmailSender {
    public function send(string $to, string $subject, string $body): bool {
        // Implémentation SendGrid
        return true;
    }
}

// ========== SRP : Service d'authentification ==========

class AuthService {
    public function __construct(
        private UserRepository $repository,
        private PasswordHasher $hasher,
        private EmailSender $emailSender
    ) {}
    
    public function register(string $email, string $password): ?User {
        // Vérifier si l'utilisateur existe déjà
        if ($this->repository->findByEmail($email) !== null) {
            throw new Exception("User already exists");
        }
        
        // Créer l'utilisateur
        $passwordHash = $this->hasher->hash($password);
        $user = new User($email, $passwordHash);
        
        // Sauvegarder
        $this->repository->save($user);
        
        // Envoyer email de confirmation
        $this->sendVerificationEmail($user);
        
        return $user;
    }
    
    public function login(string $email, string $password): ?User {
        $user = $this->repository->findByEmail($email);
        
        if ($user === null) {
            return null;
        }
        
        if (!$this->hasher->verify($password, $user->getPasswordHash())) {
            return null;
        }
        
        return $user;
    }
    
    private function sendVerificationEmail(User $user): void {
        $subject = "Verify your account";
        $body = "Please verify your account by clicking this link...";
        $this->emailSender->send($user->getEmail(), $subject, $body);
    }
}

// Utilisation
$pdo = new PDO('mysql:host=localhost;dbname=app', 'user', 'pass');
$repository = new MySQLUserRepository($pdo);
$hasher = new BcryptPasswordHasher();
$emailSender = new SMTPEmailSender();

$authService = new AuthService($repository, $hasher, $emailSender);

// Validation
$validationService = new ValidationService();
$validationService->addRule('email', new RequiredRule());
$validationService->addRule('email', new EmailRule());
$validationService->addRule('password', new RequiredRule());
$validationService->addRule('password', new PasswordStrengthRule());

$errors = $validationService->validate([
    'email' => 'user@example.com',
    'password' => 'Password123'
]);

if (empty($errors)) {
    $user = $authService->register('user@example.com', 'Password123');
}
```

## Conclusion

Ces exemples montrent comment les principes SOLID travaillent ensemble pour créer un code :
- **Maintenable** : Facile à comprendre et modifier
- **Testable** : Dépendances injectées, facile à mocker
- **Extensible** : Nouveaux types ajoutés sans modifier l'existant
- **Découplé** : Modules indépendants et réutilisables

En appliquant SOLID, vous créez une architecture solide qui évolue avec les besoins de votre application.
