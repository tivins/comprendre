# Mise en Pratique des Principes SOLID

Ce guide vous aidera à appliquer les principes SOLID dans vos projets PHP. Vous apprendrez à identifier les violations, à refactoriser progressivement et à intégrer SOLID dans votre workflow de développement.

## Comment commencer avec SOLID

### Approche progressive

Ne cherchez pas à appliquer tous les principes SOLID d'un coup. Procédez par étapes :

1. **Commencez par SRP** : C'est le plus facile à comprendre et à appliquer
2. **Ajoutez DIP** : L'injection de dépendances améliore immédiatement la testabilité
3. **Appliquez OCP** : Quand vous voyez des `if/else` répétitifs
4. **Respectez LSP** : Vérifiez que l'héritage fonctionne correctement
5. **Segmentez les interfaces** : Quand vous avez des interfaces trop larges

### Quand appliquer SOLID ?

#### Appliquez SOLID quand :

- Vous développez une nouvelle fonctionnalité
- Vous refactorisez du code existant
- Vous rencontrez des problèmes de testabilité
- Le code devient difficile à maintenir
- Plusieurs développeurs travaillent sur le même code

#### Attention à ne pas :

- Sur-appliquer sur du code simple (scripts, prototypes)
- Complexifier inutilement
- Ignorer le contexte et les contraintes du projet
- Refactoriser du code qui fonctionne sans raison valable

## Guide de refactoring

### Étape 1 : Identifier les violations

#### Signes de violation du SRP

```php
// Signes à repérer (à éviter) :
class User {
    public function save() { }           // Persistance
    public function sendEmail() { }      // Communication
    public function validate() { }        // Validation
    public function format() { }         // Formatage
    // Trop de responsabilités !
}
```

**Action** : Séparer en plusieurs classes

#### Signes de violation de l'OCP

```php
// Signes à repérer (à éviter) :
public function process($type) {
    if ($type === 'A') { }
    elseif ($type === 'B') { }
    elseif ($type === 'C') { }
    // Beaucoup de conditions = violation probable
}
```

**Action** : Utiliser le polymorphisme ou la stratégie

#### Signes de violation du LSP

```php
// Signes à repérer (à éviter) :
class Child extends Parent {
    public function method() {
        throw new Exception("Not implemented");
    }
}
```

**Action** : Vérifier que les sous-classes sont vraiment substituables

#### Signes de violation de l'ISP

```php
// Signes à repérer (à éviter) :
class Class implements LargeInterface {
    public function method1() { }
    public function method2() { }
    public function method3() { } // Méthode vide ou avec exception
    public function method4() { } // Méthode vide ou avec exception
}
```

**Action** : Séparer l'interface en plusieurs interfaces spécifiques

#### Signes de violation du DIP

```php
// Signes à repérer (à éviter) :
class Service {
    public function __construct() {
        $this->db = new MySQL(); // Instanciation directe
    }
}
```

**Action** : Injecter les dépendances via des interfaces

### Étape 2 : Refactoriser progressivement

#### Exemple : Refactoring d'une classe User

**Avant (violations multiples)** :

```php
class User {
    private $name;
    private $email;
    
    public function save() {
        $db = new PDO('mysql:...');
        $db->query("INSERT INTO users...");
    }
    
    public function sendWelcomeEmail() {
        $mailer = new PHPMailer();
        $mailer->send($this->email, "Welcome!");
    }
    
    public function validate() {
        if (empty($this->email)) {
            throw new Exception("Email required");
        }
        // ...
    }
}
```

**Après (respect de SOLID)** :

```php
// 1. SRP : Entité User simple
class User {
    private $name;
    private $email;
    
    public function getName(): string {
        return $this->name;
    }
    
    public function getEmail(): string {
        return $this->email;
    }
}

// 2. DIP : Repository abstrait
interface UserRepository {
    public function save(User $user): void;
}

class MySQLUserRepository implements UserRepository {
    public function __construct(private PDO $connection) {}
    
    public function save(User $user): void {
        // Implémentation
    }
}

// 3. SRP : Service de validation
class UserValidator {
    public function validate(User $user): array {
        $errors = [];
        if (empty($user->getEmail())) {
            $errors[] = "Email required";
        }
        return $errors;
    }
}

// 4. ISP + DIP : Service d'email
interface EmailSender {
    public function send(string $to, string $subject, string $body): bool;
}

class SMTPEmailSender implements EmailSender {
    public function send(string $to, string $subject, string $body): bool {
        // Implémentation
    }
}

// 5. SRP : Service de bienvenue
class WelcomeEmailService {
    public function __construct(private EmailSender $emailSender) {}
    
    public function sendWelcome(User $user): void {
        $this->emailSender->send(
            $user->getEmail(),
            "Welcome!",
            "Welcome to our platform!"
        );
    }
}
```

## Checklist de code review SOLID

Utilisez cette checklist lors des code reviews :

### Single Responsibility Principle (SRP)

- [ ] La classe a-t-elle une seule raison de changer ?
- [ ] Les méthodes sont-elles cohérentes avec la responsabilité de la classe ?
- [ ] Y a-t-il des méthodes qui semblent "hors sujet" ?

### Open/Closed Principle (OCP)

- [ ] Peut-on ajouter de nouvelles fonctionnalités sans modifier le code existant ?
- [ ] Y a-t-il beaucoup de `if/else` ou `switch` basés sur des types ?
- [ ] Le code utilise-t-il des abstractions (interfaces, classes abstraites) ?

### Liskov Substitution Principle (LSP)

- [ ] Les sous-classes peuvent-elles remplacer leurs classes parentes sans problème ?
- [ ] Y a-t-il des exceptions lancées dans des méthodes surchargées ?
- [ ] Les invariants de la classe parente sont-ils préservés ?

### Interface Segregation Principle (ISP)

- [ ] Les interfaces sont-elles spécifiques et ciblées ?
- [ ] Y a-t-il des méthodes vides dans les implémentations ?
- [ ] Les clients dépendent-ils seulement de ce qu'ils utilisent ?

### Dependency Inversion Principle (DIP)

- [ ] Les dépendances sont-elles injectées plutôt qu'instanciées ?
- [ ] Le code dépend-il d'abstractions plutôt que d'implémentations ?
- [ ] Les modules de haut niveau dépendent-ils des modules de bas niveau ?

## Outils et pratiques

### Outils d'analyse statique

#### PHPStan

```bash
composer require --dev phpstan/phpstan
```

Détecte certaines violations de SOLID :

```php
// PHPStan détectera cette violation du DIP
class Service {
    public function __construct() {
        $this->db = new MySQL(); // Détecté
    }
}
```

#### Psalm

```bash
composer require --dev vimeo/psalm
```

Analyse plus approfondie des dépendances et du typage.

### Tests unitaires

SOLID rend le code plus testable. Utilisez les tests pour valider vos refactorings :

```php
// Test avec mock (possible grâce au DIP)
class OrderServiceTest extends TestCase {
    public function testProcessOrder() {
        $mockRepository = $this->createMock(OrderRepository::class);
        $mockPayment = $this->createMock(PaymentGateway::class);
        
        $service = new OrderService($mockRepository, $mockPayment);
        // Test...
    }
}
```

### Design Patterns qui respectent SOLID

- **Strategy** : Respecte OCP et DIP
- **Factory** : Respecte DIP
- **Observer** : Respecte OCP et DIP
- **Repository** : Respecte DIP et SRP
- **Dependency Injection** : Respecte DIP

## Intégration dans le workflow

### 1. Code Review

Ajoutez SOLID à vos critères de review :

```markdown
## Code Review Checklist

### Fonctionnalité
- [ ] Le code fait ce qui est demandé
- [ ] Les tests passent

### Qualité (SOLID)
- [ ] SRP respecté
- [ ] OCP respecté
- [ ] LSP respecté
- [ ] ISP respecté
- [ ] DIP respecté
```

### 2. Refactoring régulier

Planifiez des sessions de refactoring :

- **Petit refactoring** : À chaque feature (15-30 min)
- **Refactoring moyen** : Mensuel (2-4 heures)
- **Grand refactoring** : Trimestriel (1-2 jours)

### 3. Formation de l'équipe

- Partagez cette documentation
- Organisez des sessions de pair programming
- Discutez des violations lors des stand-ups
- Créez des exemples de code de référence

## Cas d'usage : Refactoring complet

### Scénario : Système de facturation

**Code initial** :

```php
class Invoice {
    private $items;
    private $customer;
    
    public function calculateTotal() {
        $total = 0;
        foreach ($this->items as $item) {
            if ($this->customer->getType() === 'VIP') {
                $total += $item->getPrice() * 0.9; // 10% discount
            } elseif ($this->customer->getType() === 'PREMIUM') {
                $total += $item->getPrice() * 0.95; // 5% discount
            } else {
                $total += $item->getPrice();
            }
        }
        return $total;
    }
    
    public function save() {
        $db = new PDO('mysql:...');
        $db->query("INSERT INTO invoices...");
    }
    
    public function sendEmail() {
        $mailer = new PHPMailer();
        $mailer->send($this->customer->getEmail(), "Invoice", "...");
    }
    
    public function generatePDF() {
        $pdf = new TCPDF();
        $pdf->writeHTML("...");
        return $pdf->Output();
    }
}
```

**Violations identifiées** :
- SRP violé : Calcul, persistance, email, PDF
- OCP violé : Modification nécessaire pour nouveaux types de clients
- DIP violé : Dépendances directes à PDO, PHPMailer, TCPDF

**Refactoring étape par étape** :

#### Étape 1 : Séparer les responsabilités (SRP)

```php
class Invoice {
    private $items;
    private $customer;
    
    public function getItems(): array {
        return $this->items;
    }
    
    public function getCustomer(): Customer {
        return $this->customer;
    }
}

class InvoiceCalculator {
    public function calculateTotal(Invoice $invoice): float {
        // Logique de calcul
    }
}

class InvoiceRepository {
    public function save(Invoice $invoice): void {
        // Persistance
    }
}

class InvoiceEmailService {
    public function send(Invoice $invoice): void {
        // Envoi email
    }
}

class InvoicePDFGenerator {
    public function generate(Invoice $invoice): string {
        // Génération PDF
    }
}
```

#### Étape 2 : Appliquer OCP pour les remises

```php
interface DiscountStrategy {
    public function calculate(float $price): float;
}

class VIPDiscount implements DiscountStrategy {
    public function calculate(float $price): float {
        return $price * 0.9;
    }
}

class PremiumDiscount implements DiscountStrategy {
    public function calculate(float $price): float {
        return $price * 0.95;
    }
}

class NoDiscount implements DiscountStrategy {
    public function calculate(float $price): float {
        return $price;
    }
}

class InvoiceCalculator {
    public function __construct(private DiscountStrategy $discount) {}
    
    public function calculateTotal(Invoice $invoice): float {
        $total = 0;
        foreach ($invoice->getItems() as $item) {
            $total += $this->discount->calculate($item->getPrice());
        }
        return $total;
    }
}
```

#### Étape 3 : Appliquer DIP

```php
interface InvoiceRepository {
    public function save(Invoice $invoice): void;
}

class MySQLInvoiceRepository implements InvoiceRepository {
    public function __construct(private PDO $connection) {}
    
    public function save(Invoice $invoice): void {
        // Implémentation MySQL
    }
}

interface EmailSender {
    public function send(string $to, string $subject, string $body): bool;
}

class SMTPEmailSender implements EmailSender {
    public function send(string $to, string $subject, string $body): bool {
        // Implémentation SMTP
    }
}

class InvoiceEmailService {
    public function __construct(private EmailSender $emailSender) {}
    
    public function send(Invoice $invoice): void {
        $this->emailSender->send(
            $invoice->getCustomer()->getEmail(),
            "Invoice",
            "Your invoice..."
        );
    }
}
```

**Résultat** : Code respectant tous les principes SOLID, testable, extensible et maintenable.

## Erreurs courantes à éviter

### 1. Sur-abstraction

```php
// À éviter : Trop d'abstraction pour un cas simple
interface SimpleCalculator {
    public function add(int $a, int $b): int;
}

// Si vous n'avez qu'une seule implémentation, l'interface peut être inutile
```

### 2. Ignorer YAGNI

Ne créez pas d'abstractions "au cas où". Créez-les quand vous en avez besoin.

### 3. Refactoring prématuré

Ne refactorez pas du code qui fonctionne bien sans raison valable.

### 4. Perfectionnisme

SOLID est un guide, pas une loi absolue. Trouvez le bon équilibre.

## Métriques pour mesurer l'amélioration

### Avant/après refactoring

- **Couplage** : Nombre de dépendances (devrait diminuer)
- **Cohésion** : Liens entre méthodes d'une classe (devrait augmenter)
- **Complexité cyclomatique** : Complexité des méthodes (devrait diminuer)
- **Couverture de tests** : Pourcentage de code testé (devrait augmenter)
- **Temps de développement** : Temps pour ajouter une feature (devrait diminuer)

## Conclusion

Appliquer SOLID est un processus continu :

1. **Apprenez** : Comprenez chaque principe
2. **Identifiez** : Repérez les violations dans votre code
3. **Refactorez** : Améliorez progressivement
4. **Testez** : Validez vos changements
5. **Partagez** : Transmettez vos connaissances

Rappelez-vous : SOLID n'est pas une fin en soi, mais un moyen d'atteindre un code de qualité. L'objectif est de créer un code maintenable, testable et extensible qui facilite le travail de votre équipe.

**Commencez petit, progressez régulièrement, et n'ayez pas peur de faire des erreurs. C'est ainsi qu'on apprend !**
