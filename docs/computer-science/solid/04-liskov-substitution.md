# Liskov Substitution Principle (LSP)

## Définition

**Les objets d'une superclasse doivent pouvoir être remplacés par des objets de ses sous-classes sans casser l'application.**

Le principe de substitution de Liskov (LSP) a été formulé par Barbara Liskov en 1987. Il stipule que si `S` est un sous-type de `T`, alors les objets de type `T` peuvent être remplacés par des objets de type `S` sans altérer les propriétés souhaitables du programme.

## Comprendre LSP

### Concept de base

Si vous avez une classe `Rectangle` et une classe `Square` qui hérite de `Rectangle`, vous devriez pouvoir utiliser `Square` partout où `Rectangle` est attendu, sans que cela ne cause de problème.

### Analogie

Imaginez que vous avez une prise électrique standard :
- Vous pouvez brancher n'importe quel appareil compatible
- Si un adaptateur dit être "compatible", il doit vraiment fonctionner
- Si l'adaptateur cause des problèmes, il viole le principe de substitution

## L'exemple classique : Rectangle et Square

### Violation du LSP (à éviter)

```php
class Rectangle {
    protected $width;
    protected $height;
    
    public function setWidth($width): void {
        $this->width = $width;
    }
    
    public function setHeight($height): void {
        $this->height = $height;
    }
    
    public function getArea(): float {
        return $this->width * $this->height;
    }
}

class Square extends Rectangle {
    public function setWidth($width): void {
        $this->width = $width;
        $this->height = $width; // Comportement inattendu !
    }
    
    public function setHeight($height): void {
        $this->width = $height;
        $this->height = $height; // Comportement inattendu !
    }
}

// Problème : Le code suivant ne fonctionne pas correctement
function resizeRectangle(Rectangle $rectangle): void {
    $rectangle->setWidth(5);
    $rectangle->setHeight(4);
    // On s'attend à un rectangle 5x4 = 20
    // Mais avec Square, on obtient 4x4 = 16 !
    assert($rectangle->getArea() === 20); // Échoue avec Square
}
```

**Problème** : `Square` ne peut pas remplacer `Rectangle` car son comportement est différent. Le code qui utilise `Rectangle` s'attend à pouvoir modifier largeur et hauteur indépendamment.

### Solution : Ne pas utiliser l'héritage

```php
interface Shape {
    public function getArea(): float;
}

class Rectangle implements Shape {
    private $width;
    private $height;
    
    public function __construct($width, $height) {
        $this->width = $width;
        $this->height = $height;
    }
    
    public function getArea(): float {
        return $this->width * $this->height;
    }
}

class Square implements Shape {
    private $side;
    
    public function __construct($side) {
        $this->side = $side;
    }
    
    public function getArea(): float {
        return $this->side ** 2;
    }
}

// Maintenant, Rectangle et Square sont substituables via Shape
function printArea(Shape $shape): void {
    echo "Area: " . $shape->getArea();
}

printArea(new Rectangle(5, 4)); // Fonctionne
printArea(new Square(4));        // Fonctionne
```

## Contrats et préconditions/postconditions

### Préconditions

Les sous-classes ne doivent **pas renforcer** les préconditions (conditions requises avant l'exécution).

```php
// À éviter : Précondition renforcée
class Bird {
    public function fly(): void {
        // Tous les oiseaux peuvent voler
    }
}

class Penguin extends Bird {
    public function fly(): void {
        throw new Exception("Penguins can't fly!"); // Précondition renforcée
    }
}

function makeBirdFly(Bird $bird): void {
    $bird->fly(); // Échoue avec Penguin
}

// Solution : Ne pas hériter si le comportement diffère
interface Flyable {
    public function fly(): void;
}

class Bird implements Flyable {
    public function fly(): void {
        // Voler
    }
}

class Penguin {
    // Pas d'interface Flyable, car les pingouins ne volent pas
    public function swim(): void {
        // Nager
    }
}
```

### Postconditions

Les sous-classes ne doivent **pas affaiblir** les postconditions (garanties après l'exécution).

```php
// À éviter : Postcondition affaiblie
class Database {
    public function save($data): bool {
        // Sauvegarde et retourne toujours true
        return true;
    }
}

class UnreliableDatabase extends Database {
    public function save($data): bool {
        // Parfois retourne false même si la sauvegarde réussit
        return rand(0, 1) === 1; // Postcondition affaiblie
    }
}

function saveData(Database $db, $data): void {
    $result = $db->save($data);
    if (!$result) {
        throw new Exception("Save failed");
    }
    // Le code s'attend à ce que save() retourne true si réussi
    // Mais UnreliableDatabase peut retourner false même si réussi
}

// Solution : Respecter le contrat
class ReliableDatabase extends Database {
    public function save($data): bool {
        try {
            // Sauvegarde
            return true; // Toujours true si réussi
        } catch (Exception $e) {
            return false; // False seulement en cas d'erreur
        }
    }
}
```

## Exemples détaillés

### Exemple 1 : Collections

```php
// À éviter : Comportement différent
class ReadOnlyList {
    protected $items = [];
    
    public function add($item): void {
        $this->items[] = $item;
    }
    
    public function get($index) {
        return $this->items[$index];
    }
}

class ImmutableList extends ReadOnlyList {
    public function add($item): void {
        throw new Exception("Cannot add to immutable list"); // Violation LSP
    }
}

function populateList(ReadOnlyList $list): void {
    $list->add('item1'); // Échoue avec ImmutableList
    $list->add('item2');
}

// Solution : Interfaces séparées
interface Readable {
    public function get($index);
}

interface Writable {
    public function add($item): void;
}

class MutableList implements Readable, Writable {
    private $items = [];
    
    public function add($item): void {
        $this->items[] = $item;
    }
    
    public function get($index) {
        return $this->items[$index];
    }
}

class ImmutableList implements Readable {
    private $items;
    
    public function __construct(array $items) {
        $this->items = $items;
    }
    
    public function get($index) {
        return $this->items[$index];
    }
}

function populateList(Writable $list): void {
    $list->add('item1'); // Fonctionne seulement avec MutableList
    $list->add('item2');
}
```

### Exemple 2 : Paiements

```php
// À éviter : Comportement différent
abstract class PaymentMethod {
    abstract public function pay(float $amount): bool;
    
    public function refund(float $amount): bool {
        // Par défaut, tous les paiements peuvent être remboursés
        return true;
    }
}

class CreditCardPayment extends PaymentMethod {
    public function pay(float $amount): bool {
        // Paiement par carte
        return true;
    }
    
    public function refund(float $amount): bool {
        // Remboursement possible
        return true;
    }
}

class CashPayment extends PaymentMethod {
    public function pay(float $amount): bool {
        // Paiement en espèces
        return true;
    }
    
    public function refund(float $amount): bool {
        // Les espèces ne peuvent pas être remboursées de la même manière
        throw new Exception("Cash cannot be refunded"); // Violation LSP
    }
}

function processRefund(PaymentMethod $payment, float $amount): void {
    $payment->refund($amount); // Échoue avec CashPayment
}

// Solution : Interfaces séparées
interface Payable {
    public function pay(float $amount): bool;
}

interface Refundable {
    public function refund(float $amount): bool;
}

class CreditCardPayment implements Payable, Refundable {
    public function pay(float $amount): bool {
        return true;
    }
    
    public function refund(float $amount): bool {
        return true;
    }
}

class CashPayment implements Payable {
    public function pay(float $amount): bool {
        return true;
    }
    // Pas d'interface Refundable
}

function processRefund(Refundable $payment, float $amount): void {
    $payment->refund($amount); // Fonctionne seulement avec les paiements remboursables
}
```

### Exemple 3 : Validation

```php
// À éviter : Contrat différent
abstract class Validator {
    abstract public function validate($value): bool;
    
    public function getErrorMessage(): string {
        return "Validation failed";
    }
}

class EmailValidator extends Validator {
    public function validate($value): bool {
        return filter_var($value, FILTER_VALIDATE_EMAIL) !== false;
    }
}

class RequiredValidator extends Validator {
    private $errorMessage;
    
    public function validate($value): bool {
        $result = !empty($value);
        if (!$result) {
            $this->errorMessage = "Field is required"; // État modifié différemment
        }
        return $result;
    }
    
    public function getErrorMessage(): string {
        return $this->errorMessage ?? parent::getErrorMessage();
    }
}

function validateValue(Validator $validator, $value): void {
    if (!$validator->validate($value)) {
        echo $validator->getErrorMessage(); // Comportement différent selon la classe
    }
}

// Solution : Contrat cohérent
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

class EmailValidator implements Validator {
    public function validate($value): ValidationResult {
        $isValid = filter_var($value, FILTER_VALIDATE_EMAIL) !== false;
        return new ValidationResult($isValid, $isValid ? '' : 'Invalid email');
    }
}

class RequiredValidator implements Validator {
    public function validate($value): ValidationResult {
        $isValid = !empty($value);
        return new ValidationResult($isValid, $isValid ? '' : 'Field is required');
    }
}

function validateValue(Validator $validator, $value): void {
    $result = $validator->validate($value);
    if (!$result->isValid()) {
        echo $result->getErrorMessage(); // Comportement cohérent
    }
}
```

## Comment détecter les violations du LSP

### Signes de violation

1. **Exceptions dans les méthodes surchargées** : Si une sous-classe lance une exception là où la classe parente ne le fait pas
2. **Retours null inattendus** : Si une sous-classe retourne null là où la classe parente retourne toujours une valeur
3. **Comportement conditionnel** : Si vous devez vérifier le type avant d'utiliser l'objet
4. **Tests qui échouent** : Si les tests de la classe parente échouent avec la sous-classe

### Test de substitution

```php
// Si ce test passe pour la classe parente mais échoue pour la sous-classe,
// c'est une violation du LSP
function testSubstitution(ParentClass $obj): void {
    // Tests qui doivent fonctionner pour toutes les sous-classes
    assert($obj->method() !== null);
    assert($obj->method() instanceof ExpectedType);
    // ...
}
```

## Avantages du LSP

1. **Fiabilité** : Le code fonctionne de manière prévisible avec toutes les sous-classes
2. **Testabilité** : Les tests de la classe parente fonctionnent avec les sous-classes
3. **Maintenabilité** : Moins de surprises, comportement cohérent
4. **Réutilisabilité** : Le code qui utilise la classe parente fonctionne avec toutes les sous-classes
5. **Polymorphisme fiable** : Le polymorphisme fonctionne comme prévu

## Pièges à éviter

### 1. Héritage "est-un" vs "se comporte comme"

```php
// À éviter : "Square est un Rectangle" mais ne se comporte pas comme un Rectangle
class Square extends Rectangle { }

// Bon : "Square se comporte comme une Shape"
class Square implements Shape { }
```

### 2. Violation des invariants

```php
// À éviter : Invariant violé
class Stack {
    protected $items = [];
    
    public function push($item): void {
        array_push($this->items, $item);
    }
    
    public function pop() {
        return array_pop($this->items);
    }
}

class Queue extends Stack {
    public function push($item): void {
        array_unshift($this->items, $item); // Comportement différent
    }
}

// Stack : dernier entré, premier sorti (LIFO)
// Queue : premier entré, premier sorti (FIFO)
// Queue ne peut pas remplacer Stack
```

### 3. Ignorer les contrats implicites

Même sans interface explicite, il y a un contrat implicite. Respectez-le.

## Comment appliquer le LSP

1. **Respecter les contrats** : Les sous-classes doivent respecter le contrat de la classe parente
2. **Ne pas renforcer les préconditions** : Ne pas rendre les conditions d'entrée plus strictes
3. **Ne pas affaiblir les postconditions** : Ne pas réduire les garanties de sortie
4. **Préserver les invariants** : Maintenir les propriétés qui doivent toujours être vraies
5. **Tester la substitution** : Les tests de la classe parente doivent fonctionner avec les sous-classes

## Conclusion

Le principe de substitution de Liskov garantit que l'héritage fonctionne correctement. Les sous-classes doivent être véritablement substituables à leurs classes parentes sans casser le comportement attendu. Si une sous-classe ne peut pas remplacer sa classe parente sans problème, c'est probablement qu'elle ne devrait pas hériter de cette classe. **Les sous-classes doivent être de vrais substituts**.
