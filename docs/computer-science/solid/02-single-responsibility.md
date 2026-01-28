# Single Responsibility Principle (SRP)

## Définition

**Une classe ne devrait avoir qu'une seule raison de changer.**

Le principe de responsabilité unique (SRP) est le premier principe SOLID. Il stipule qu'une classe ne devrait avoir qu'une seule responsabilité, c'est-à-dire qu'elle ne devrait avoir qu'une seule raison d'être modifiée.

## Comprendre la "responsabilité"

Une **responsabilité** est une raison de changer. Si une classe a plusieurs raisons de changer, elle a plusieurs responsabilités et viole le SRP.

### Exemple de responsabilités multiples

```php
// ❌ Violation du SRP : Trois responsabilités
class User {
    // Responsabilité 1 : Gestion des données utilisateur
    private $name;
    private $email;
    
    public function getName() { return $this->name; }
    public function getEmail() { return $this->email; }
    
    // Responsabilité 2 : Persistance en base de données
    public function save() {
        $db = new Database();
        $db->query("INSERT INTO users ...");
    }
    
    // Responsabilité 3 : Envoi d'emails
    public function sendWelcomeEmail() {
        $mailer = new Mailer();
        $mailer->send($this->email, "Welcome!");
    }
}
```

**Problèmes** :
- Si la structure de la base de données change → modification de `User`
- Si le service d'email change → modification de `User`
- Si les règles métier changent → modification de `User`
- Difficile à tester (besoin de mocker la BDD et le mailer)
- Difficile à réutiliser (couplé à la BDD et au mailer)

## Solution : Séparer les responsabilités

```php
// ✅ Respect du SRP : Une responsabilité par classe

// Responsabilité 1 : Représentation des données utilisateur
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

// Responsabilité 2 : Persistance
class UserRepository {
    public function save(User $user): void {
        // Logique de sauvegarde
    }
    
    public function findById(int $id): ?User {
        // Logique de récupération
    }
}

// Responsabilité 3 : Envoi d'emails
class EmailService {
    public function sendWelcomeEmail(User $user): void {
        // Logique d'envoi d'email
    }
}
```

## Comment identifier les responsabilités ?

### Questions à se poser

1. **Quelles sont les raisons de changer cette classe ?**
   - Changement de structure de données ?
   - Changement de logique métier ?
   - Changement de persistance ?
   - Changement d'affichage ?

2. **Cette classe fait-elle plusieurs choses différentes ?**
   - Gère-t-elle des données ET les sauvegarde ?
   - Calcule-t-elle ET affiche-t-elle ?
   - Valide-t-elle ET transforme-t-elle ?

3. **Si je dois modifier X, dois-je toucher cette classe ?**
   - Si oui, cette classe a une responsabilité liée à X
   - Si plusieurs X nécessitent des modifications → plusieurs responsabilités

## Exemples détaillés

### Exemple 1 : Gestionnaire de commandes

```php
// ❌ Violation : La classe fait trop de choses
class Order {
    private $items;
    private $total;
    
    public function calculateTotal() {
        // Calcul du total
    }
    
    public function saveToDatabase() {
        // Sauvegarde en BDD
    }
    
    public function sendConfirmationEmail() {
        // Envoi d'email
    }
    
    public function generateInvoice() {
        // Génération de facture PDF
    }
    
    public function printReceipt() {
        // Impression du reçu
    }
}

// ✅ Respect : Séparation des responsabilités
class Order {
    private $items;
    private $total;
    
    public function calculateTotal(): float {
        // Seulement le calcul
    }
    
    public function getItems(): array {
        return $this->items;
    }
}

class OrderRepository {
    public function save(Order $order): void {
        // Sauvegarde en BDD
    }
}

class OrderEmailService {
    public function sendConfirmation(Order $order): void {
        // Envoi d'email
    }
}

class InvoiceGenerator {
    public function generate(Order $order): string {
        // Génération PDF
    }
}

class ReceiptPrinter {
    public function print(Order $order): void {
        // Impression
    }
}
```

### Exemple 2 : Validation et transformation

```php
// ❌ Violation : Validation ET transformation
class UserDataProcessor {
    public function process(array $data): User {
        // Validation
        if (empty($data['email'])) {
            throw new Exception("Email required");
        }
        if (!filter_var($data['email'], FILTER_VALIDATE_EMAIL)) {
            throw new Exception("Invalid email");
        }
        
        // Transformation
        $user = new User();
        $user->setName(strtoupper($data['name']));
        $user->setEmail(strtolower($data['email']));
        
        return $user;
    }
}

// ✅ Respect : Séparation validation/transformation
class UserValidator {
    public function validate(array $data): void {
        if (empty($data['email'])) {
            throw new ValidationException("Email required");
        }
        if (!filter_var($data['email'], FILTER_VALIDATE_EMAIL)) {
            throw new ValidationException("Invalid email");
        }
    }
}

class UserFactory {
    public function createFromArray(array $data): User {
        $user = new User();
        $user->setName(strtoupper($data['name']));
        $user->setEmail(strtolower($data['email']));
        return $user;
    }
}

// Utilisation
$validator = new UserValidator();
$validator->validate($data);

$factory = new UserFactory();
$user = $factory->createFromArray($data);
```

### Exemple 3 : Rapport et formatage

```php
// ❌ Violation : Génération ET formatage
class ReportGenerator {
    public function generate(array $data): string {
        // Collecte des données
        $report = [];
        foreach ($data as $item) {
            $report[] = [
                'date' => $item->getDate(),
                'amount' => $item->getAmount(),
            ];
        }
        
        // Formatage HTML
        $html = '<table>';
        foreach ($report as $row) {
            $html .= '<tr>';
            $html .= '<td>' . htmlspecialchars($row['date']) . '</td>';
            $html .= '<td>' . htmlspecialchars($row['amount']) . '</td>';
            $html .= '</tr>';
        }
        $html .= '</table>';
        
        return $html;
    }
}

// ✅ Respect : Séparation génération/formatage
class ReportDataCollector {
    public function collect(array $data): array {
        $report = [];
        foreach ($data as $item) {
            $report[] = [
                'date' => $item->getDate(),
                'amount' => $item->getAmount(),
            ];
        }
        return $report;
    }
}

class HtmlFormatter {
    public function format(array $data): string {
        $html = '<table>';
        foreach ($data as $row) {
            $html .= '<tr>';
            $html .= '<td>' . htmlspecialchars($row['date']) . '</td>';
            $html .= '<td>' . htmlspecialchars($row['amount']) . '</td>';
            $html .= '</tr>';
        }
        $html .= '</table>';
        return $html;
    }
}

class JsonFormatter {
    public function format(array $data): string {
        return json_encode($data);
    }
}

// Utilisation flexible
$collector = new ReportDataCollector();
$data = $collector->collect($rawData);

$formatter = new HtmlFormatter(); // ou JsonFormatter
$output = $formatter->format($data);
```

## SRP et les méthodes

Le SRP s'applique aussi aux méthodes. Une méthode devrait faire une seule chose.

```php
// ❌ Violation : La méthode fait plusieurs choses
public function processOrder(Order $order) {
    // 1. Valider
    if (!$order->isValid()) {
        throw new Exception("Invalid order");
    }
    
    // 2. Calculer
    $total = $order->calculateTotal();
    
    // 3. Sauvegarder
    $this->repository->save($order);
    
    // 4. Envoyer email
    $this->emailService->send($order);
    
    // 5. Logger
    $this->logger->info("Order processed");
}

// ✅ Respect : Méthode orchestratrice qui délègue
public function processOrder(Order $order): void {
    $this->validateOrder($order);
    $this->saveOrder($order);
    $this->notifyCustomer($order);
    $this->logOrder($order);
}

private function validateOrder(Order $order): void {
    if (!$order->isValid()) {
        throw new ValidationException("Invalid order");
    }
}

private function saveOrder(Order $order): void {
    $this->repository->save($order);
}

private function notifyCustomer(Order $order): void {
    $this->emailService->send($order);
}

private function logOrder(Order $order): void {
    $this->logger->info("Order processed", ['order_id' => $order->getId()]);
}
```

## Avantages du SRP

1. **Testabilité** : Classes avec une seule responsabilité sont plus faciles à tester
2. **Maintenabilité** : Modifications localisées, moins de risques de régression
3. **Réutilisabilité** : Classes découplées peuvent être réutilisées
4. **Compréhension** : Code plus clair et plus facile à comprendre
5. **Collaboration** : Plusieurs développeurs peuvent travailler sans conflits

## Pièges à éviter

### 1. Sur-séparation

```php
// ❌ Trop de classes pour une responsabilité simple
class UserNameGetter {
    public function getName(User $user): string {
        return $user->getName();
    }
}

// ✅ Inutile, User peut avoir getName() directement
class User {
    public function getName(): string {
        return $this->name;
    }
}
```

### 2. Confusion responsabilité / méthode

```php
// ✅ OK : Une classe peut avoir plusieurs méthodes
class User {
    public function getName(): string { }
    public function getEmail(): string { }
    public function getAge(): int { }
    // Toutes ces méthodes concernent la même responsabilité : représenter un User
}
```

### 3. Ignorer le contexte

Dans un petit script ou prototype, une violation mineure du SRP peut être acceptable. L'important est de reconnaître quand refactoriser.

## Comment appliquer le SRP

1. **Identifier** les responsabilités dans votre classe
2. **Séparer** chaque responsabilité dans sa propre classe
3. **Découpler** les classes avec des dépendances explicites
4. **Tester** chaque classe indépendamment
5. **Refactoriser** progressivement si nécessaire

## Conclusion

Le Single Responsibility Principle est le fondement des autres principes SOLID. Un code qui respecte le SRP est plus facile à comprendre, tester et maintenir. Rappelez-vous : **une classe, une responsabilité, une raison de changer**.
