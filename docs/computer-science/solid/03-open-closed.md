# Open/Closed Principle (OCP)

## Définition

**Les entités logicielles (classes, modules, fonctions) devraient être ouvertes à l'extension mais fermées à la modification.**

Le principe ouvert/fermé (OCP) signifie que vous devriez pouvoir ajouter de nouvelles fonctionnalités sans modifier le code existant qui fonctionne déjà.

## Comprendre OCP

### "Ouvert à l'extension"
Vous pouvez ajouter de nouveaux comportements ou fonctionnalités.

### "Fermé à la modification"
Vous ne devez pas modifier le code existant qui fonctionne déjà.

### Analogie

Imaginez une prise électrique :
- **Fermée à la modification** : Vous ne modifiez pas la prise elle-même
- **Ouverte à l'extension** : Vous pouvez brancher n'importe quel appareil compatible

## Pourquoi OCP est important ?

### Problèmes sans OCP

```php
// ❌ Violation : Modification nécessaire pour chaque nouveau type
class AreaCalculator {
    public function calculate($shape) {
        if ($shape instanceof Circle) {
            return pi() * $shape->getRadius() ** 2;
        }
        
        if ($shape instanceof Rectangle) {
            return $shape->getWidth() * $shape->getHeight();
        }
        
        // Pour ajouter Triangle, il faut modifier cette classe !
        // Risque de casser le code existant
    }
}
```

**Problèmes** :
- Chaque nouveau type nécessite une modification
- Risque de régression sur le code existant
- Violation du SRP (la classe connaît tous les types)
- Difficile à tester (besoin de tester tous les cas)

## Solution : Utiliser l'abstraction

```php
// ✅ Respect : Extension sans modification
interface Shape {
    public function area(): float;
}

class Circle implements Shape {
    private float $radius;
    
    public function area(): float {
        return pi() * $this->radius ** 2;
    }
}

class Rectangle implements Shape {
    private float $width;
    private float $height;
    
    public function area(): float {
        return $this->width * $this->height;
    }
}

class AreaCalculator {
    public function calculate(Shape $shape): float {
        // Pas besoin de connaître le type concret !
        return $shape->area();
    }
}

// Extension : Ajouter Triangle sans modifier AreaCalculator
class Triangle implements Shape {
    private float $base;
    private float $height;
    
    public function area(): float {
        return ($this->base * $this->height) / 2;
    }
}
```

## Stratégies pour respecter OCP

### 1. Utiliser des interfaces et le polymorphisme

```php
// Exemple : Système de paiement
interface PaymentProcessor {
    public function process(float $amount): bool;
}

class CreditCardProcessor implements PaymentProcessor {
    public function process(float $amount): bool {
        // Logique de paiement par carte
        return true;
    }
}

class PayPalProcessor implements PaymentProcessor {
    public function process(float $amount): bool {
        // Logique PayPal
        return true;
    }
}

class PaymentService {
    public function __construct(private PaymentProcessor $processor) {}
    
    public function pay(float $amount): bool {
        return $this->processor->process($amount);
    }
}

// Extension : Ajouter Stripe sans modifier PaymentService
class StripeProcessor implements PaymentProcessor {
    public function process(float $amount): bool {
        // Logique Stripe
        return true;
    }
}
```

### 2. Utiliser l'héritage avec des classes abstraites

```php
abstract class Report {
    abstract public function generate(): string;
    
    // Méthode commune (template method)
    public function render(): string {
        $content = $this->generate();
        return $this->format($content);
    }
    
    protected function format(string $content): string {
        return "<html><body>{$content}</body></html>";
    }
}

class SalesReport extends Report {
    public function generate(): string {
        return "Sales: $1000";
    }
}

class InventoryReport extends Report {
    public function generate(): string {
        return "Items: 50";
    }
}

// Extension : Ajouter un nouveau type de rapport
class FinancialReport extends Report {
    public function generate(): string {
        return "Profit: $5000";
    }
}
```

### 3. Utiliser la composition et la stratégie

```php
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

class PriceCalculator {
    public function __construct(private DiscountStrategy $discount) {}
    
    public function calculate(float $basePrice): float {
        return $this->discount->calculate($basePrice);
    }
}

// Extension : Ajouter un nouveau type de réduction
class SeasonalDiscount implements DiscountStrategy {
    public function calculate(float $price): float {
        // Logique de réduction saisonnière
        return $price * 0.8;
    }
}
```

## Exemples détaillés

### Exemple 1 : Filtrage de produits

```php
// ❌ Violation : Modification nécessaire pour chaque critère
class ProductFilter {
    public function filterByColor(array $products, string $color): array {
        return array_filter($products, fn($p) => $p->getColor() === $color);
    }
    
    public function filterBySize(array $products, string $size): array {
        return array_filter($products, fn($p) => $p->getSize() === $size);
    }
    
    // Pour ajouter filterByPrice, il faut modifier cette classe
}

// ✅ Respect : Extension via interface
interface Specification {
    public function isSatisfiedBy(Product $product): bool;
}

class ColorSpecification implements Specification {
    public function __construct(private string $color) {}
    
    public function isSatisfiedBy(Product $product): bool {
        return $product->getColor() === $this->color;
    }
}

class SizeSpecification implements Specification {
    public function __construct(private string $size) {}
    
    public function isSatisfiedBy(Product $product): bool {
        return $product->getSize() === $this->size;
    }
}

class ProductFilter {
    public function filter(array $products, Specification $spec): array {
        return array_filter($products, fn($p) => $spec->isSatisfiedBy($p));
    }
}

// Extension : Ajouter PriceSpecification sans modifier ProductFilter
class PriceSpecification implements Specification {
    public function __construct(private float $minPrice, private float $maxPrice) {}
    
    public function isSatisfiedBy(Product $product): bool {
        $price = $product->getPrice();
        return $price >= $this->minPrice && $price <= $this->maxPrice;
    }
}

// Utilisation
$filter = new ProductFilter();
$redProducts = $filter->filter($products, new ColorSpecification('red'));
$cheapProducts = $filter->filter($products, new PriceSpecification(0, 50));
```

### Exemple 2 : Génération de rapports

```php
// ❌ Violation : Modification nécessaire pour chaque format
class ReportGenerator {
    public function generate(array $data, string $format): string {
        if ($format === 'html') {
            return $this->generateHtml($data);
        }
        
        if ($format === 'json') {
            return $this->generateJson($data);
        }
        
        // Pour ajouter 'xml', il faut modifier cette classe
    }
    
    private function generateHtml(array $data): string {
        // ...
    }
    
    private function generateJson(array $data): string {
        // ...
    }
}

// ✅ Respect : Extension via interface
interface ReportFormatter {
    public function format(array $data): string;
}

class HtmlFormatter implements ReportFormatter {
    public function format(array $data): string {
        $html = '<table>';
        foreach ($data as $row) {
            $html .= '<tr><td>' . htmlspecialchars($row) . '</td></tr>';
        }
        $html .= '</table>';
        return $html;
    }
}

class JsonFormatter implements ReportFormatter {
    public function format(array $data): string {
        return json_encode($data);
    }
}

class ReportGenerator {
    public function generate(array $data, ReportFormatter $formatter): string {
        return $formatter->format($data);
    }
}

// Extension : Ajouter XmlFormatter sans modifier ReportGenerator
class XmlFormatter implements ReportFormatter {
    public function format(array $data): string {
        $xml = '<report>';
        foreach ($data as $row) {
            $xml .= '<item>' . htmlspecialchars($row) . '</item>';
        }
        $xml .= '</report>';
        return $xml;
    }
}
```

### Exemple 3 : Validation de données

```php
// ❌ Violation : Modification nécessaire pour chaque règle
class UserValidator {
    public function validate(array $data): array {
        $errors = [];
        
        if (empty($data['email'])) {
            $errors[] = 'Email is required';
        }
        
        if (!filter_var($data['email'], FILTER_VALIDATE_EMAIL)) {
            $errors[] = 'Email is invalid';
        }
        
        if (strlen($data['password']) < 8) {
            $errors[] = 'Password must be at least 8 characters';
        }
        
        // Pour ajouter une nouvelle règle, il faut modifier cette classe
        return $errors;
    }
}

// ✅ Respect : Extension via interface
interface ValidationRule {
    public function validate($value): ?string; // Retourne null si valide, message d'erreur sinon
}

class RequiredRule implements ValidationRule {
    public function validate($value): ?string {
        return empty($value) ? 'Field is required' : null;
    }
}

class EmailRule implements ValidationRule {
    public function validate($value): ?string {
        return filter_var($value, FILTER_VALIDATE_EMAIL) ? null : 'Email is invalid';
    }
}

class MinLengthRule implements ValidationRule {
    public function __construct(private int $minLength) {}
    
    public function validate($value): ?string {
        return strlen($value) >= $this->minLength 
            ? null 
            : "Must be at least {$this->minLength} characters";
    }
}

class Validator {
    private array $rules = [];
    
    public function addRule(string $field, ValidationRule $rule): void {
        $this->rules[$field][] = $rule;
    }
    
    public function validate(array $data): array {
        $errors = [];
        
        foreach ($this->rules as $field => $fieldRules) {
            $value = $data[$field] ?? null;
            
            foreach ($fieldRules as $rule) {
                $error = $rule->validate($value);
                if ($error !== null) {
                    $errors[$field][] = $error;
                }
            }
        }
        
        return $errors;
    }
}

// Extension : Ajouter de nouvelles règles sans modifier Validator
class RegexRule implements ValidationRule {
    public function __construct(private string $pattern, private string $message) {}
    
    public function validate($value): ?string {
        return preg_match($this->pattern, $value) ? null : $this->message;
    }
}

// Utilisation
$validator = new Validator();
$validator->addRule('email', new RequiredRule());
$validator->addRule('email', new EmailRule());
$validator->addRule('password', new RequiredRule());
$validator->addRule('password', new MinLengthRule(8));
$validator->addRule('phone', new RegexRule('/^\+33\d{9}$/', 'Invalid phone number'));
```

## OCP et les conditions

Les structures conditionnelles (`if/else`, `switch`) sont souvent des signes de violation de l'OCP.

```php
// ❌ Beaucoup de conditions = violation probable de l'OCP
public function process($type, $data) {
    if ($type === 'A') {
        // ...
    } elseif ($type === 'B') {
        // ...
    } elseif ($type === 'C') {
        // ...
    }
    // Chaque nouveau type nécessite une modification
}
```

**Solution** : Utiliser le polymorphisme ou la stratégie.

## Avantages de l'OCP

1. **Stabilité** : Le code existant n'est pas modifié, donc moins de risques de régression
2. **Extensibilité** : Facile d'ajouter de nouvelles fonctionnalités
3. **Testabilité** : Chaque extension peut être testée indépendamment
4. **Maintenabilité** : Modifications localisées, code plus clair
5. **Réutilisabilité** : Composants réutilisables via interfaces

## Pièges à éviter

### 1. Sur-abstraction

```php
// ❌ Trop d'abstraction pour un cas simple
interface SimpleCalculator {
    public function add(int $a, int $b): int;
}

// Si vous n'avez qu'une seule implémentation, l'interface peut être inutile
```

### 2. Ignorer YAGNI (You Aren't Gonna Need It)

Ne créez pas d'abstractions "au cas où". Créez-les quand vous avez besoin d'étendre.

### 3. Confondre extension et modification

```php
// ❌ Ce n'est pas une extension, c'est une modification
class Calculator {
    public function add(int $a, int $b): int {
        return $a + $b;
    }
    
    // Ajouter une méthode = modification, pas extension
    public function subtract(int $a, int $b): int {
        return $a - $b;
    }
}
```

## Comment appliquer l'OCP

1. **Identifier** les endroits où vous ajoutez souvent des `if/else`
2. **Extraire** les variations dans des interfaces ou classes abstraites
3. **Utiliser** le polymorphisme pour gérer les variations
4. **Tester** que les extensions fonctionnent sans modifier l'existant
5. **Refactoriser** progressivement quand vous voyez des patterns répétitifs

## Conclusion

Le principe ouvert/fermé permet de créer des systèmes extensibles sans risquer de casser le code existant. En utilisant des abstractions (interfaces, classes abstraites), vous pouvez ajouter de nouvelles fonctionnalités en créant de nouvelles implémentations plutôt qu'en modifiant le code existant. **Étendez, ne modifiez pas**.
