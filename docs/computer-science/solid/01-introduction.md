# Introduction aux principes SOLID

## Qu'est-ce que SOLID ?

SOLID est un acronyme représentant cinq principes fondamentaux de la conception orientée objet. Ces principes ont été formulés par Robert C. Martin (surnommé "Uncle Bob"), l'un des signataires du Manifeste Agile et auteur de nombreux livres sur le développement logiciel.

### Origine et historique

Les principes SOLID ont été introduits dans les années 2000, mais leurs concepts remontent aux travaux de :
- **Barbara Liskov** (1987) - Principe de substitution
- **Bertrand Meyer** (1988) - Principe ouvert/fermé dans son livre "Object-Oriented Software Construction"
- **Robert C. Martin** (2000) - Consolidation et popularisation des principes

Le terme "SOLID" lui-même a été créé par Michael Feathers pour faciliter la mémorisation de ces principes.

## Pourquoi SOLID existe-t-il ?

### Le problème du code "spaghetti"

Sans principes de conception clairs, le code tend naturellement vers :
- **Couplage fort** : Les classes dépendent directement les unes des autres
- **Responsabilités multiples** : Une classe fait trop de choses différentes
- **Rigidité** : Modifier une partie du code casse d'autres parties
- **Fragilité** : Les bugs apparaissent dans des endroits inattendus
- **Impossibilité de réutiliser** : Le code est trop spécifique et couplé

### La solution : SOLID

SOLID fournit des règles claires pour :
- **Découpler** les composants
- **Séparer** les responsabilités
- **Faciliter** l'extension sans modification
- **Assurer** la substituabilité
- **Réduire** les dépendances inutiles

## Les cinq principes en bref

### S - Single Responsibility Principle (SRP)
**Une classe ne devrait avoir qu'une seule raison de changer.**

Si une classe a plusieurs responsabilités, elle devient fragile. Chaque changement peut affecter plusieurs fonctionnalités.

**Exemple simple** :
```php
// À éviter : La classe fait deux choses
class User {
    public function save() { /* sauvegarde en BDD */ }
    public function sendEmail() { /* envoie un email */ }
}

// Bon : Séparation des responsabilités
class User {
    public function save() { /* sauvegarde en BDD */ }
}
class EmailService {
    public function send(User $user) { /* envoie un email */ }
}
```

### O - Open/Closed Principle (OCP)
**Ouvert à l'extension, fermé à la modification.**

Vous devriez pouvoir ajouter de nouvelles fonctionnalités sans modifier le code existant.

**Exemple simple** :
```php
// À éviter : Modification nécessaire pour ajouter un type
class AreaCalculator {
    public function calculate($shape) {
        if ($shape instanceof Circle) {
            return pi() * $shape->radius ** 2;
        }
        if ($shape instanceof Rectangle) {
            return $shape->width * $shape->height;
        }
        // Il faut modifier cette classe pour ajouter un Triangle
    }
}

// Bon : Extension sans modification
interface Shape {
    public function area();
}
class Circle implements Shape {
    public function area() { return pi() * $this->radius ** 2; }
}
class Rectangle implements Shape {
    public function area() { return $this->width * $this->height; }
}
// On peut ajouter Triangle sans modifier AreaCalculator
```

### L - Liskov Substitution Principle (LSP)
**Les sous-classes doivent pouvoir remplacer leurs classes parentes.**

Si vous avez une classe `B` qui hérite de `A`, vous devriez pouvoir utiliser `B` partout où `A` est attendu sans casser le comportement.

**Exemple simple** :
```php
// À éviter : Rectangle et Square ne sont pas substituables
class Rectangle {
    public function setWidth($w) { $this->width = $w; }
    public function setHeight($h) { $this->height = $h; }
}
class Square extends Rectangle {
    public function setWidth($w) {
        $this->width = $w;
        $this->height = $w; // Comportement inattendu !
    }
}

// Bon : Comportement cohérent
interface Shape {
    public function area();
}
class Rectangle implements Shape {
    public function area() { return $this->width * $this->height; }
}
class Square implements Shape {
    public function area() { return $this->side ** 2; }
}
```

### I - Interface Segregation Principle (ISP)
**Ne forcez pas les clients à dépendre d'interfaces qu'ils n'utilisent pas.**

Mieux vaut plusieurs interfaces spécifiques qu'une seule interface générale.

**Exemple simple** :
```php
// À éviter : Interface trop large
interface Worker {
    public function work();
    public function eat();
    public function sleep();
}
class Robot implements Worker {
    public function work() { /* OK */ }
    public function eat() { /* Robot ne mange pas ! */ }
    public function sleep() { /* Robot ne dort pas ! */ }
}

// Bon : Interfaces séparées
interface Workable {
    public function work();
}
interface Eatable {
    public function eat();
}
class Robot implements Workable {
    public function work() { /* OK */ }
}
class Human implements Workable, Eatable {
    public function work() { /* OK */ }
    public function eat() { /* OK */ }
}
```

### D - Dependency Inversion Principle (DIP)
**Dépendre des abstractions, pas des implémentations concrètes.**

Les modules de haut niveau ne devraient pas dépendre des modules de bas niveau. Les deux devraient dépendre d'abstractions.

**Exemple simple** :
```php
// À éviter : Dépendance directe à MySQL
class UserRepository {
    private $mysql;
    public function __construct() {
        $this->mysql = new MySQLConnection();
    }
}

// Bon : Dépendance à une abstraction
interface DatabaseConnection {
    public function query($sql);
}
class UserRepository {
    private $db;
    public function __construct(DatabaseConnection $db) {
        $this->db = $db; // Peut être MySQL, PostgreSQL, etc.
    }
}
```

## Comment SOLID s'articule ensemble

Les principes SOLID ne sont pas indépendants. Ils travaillent ensemble :

1. **SRP** crée des classes avec une seule responsabilité
2. **OCP** permet d'étendre ces classes sans les modifier
3. **LSP** assure que l'héritage fonctionne correctement
4. **ISP** garantit que les interfaces sont appropriées
5. **DIP** découple les dépendances grâce aux abstractions

### Exemple d'interaction

```php
// SRP : Chaque classe a une responsabilité
interface PaymentProcessor { // ISP : Interface spécifique
    public function process(Payment $payment);
}

class StripeProcessor implements PaymentProcessor { // LSP : Substituable
    public function process(Payment $payment) { /* ... */ }
}

class PayPalProcessor implements PaymentProcessor { // OCP : Extension sans modification
    public function process(Payment $payment) { /* ... */ }
}

class OrderService {
    // DIP : Dépend de l'abstraction, pas de l'implémentation
    public function __construct(private PaymentProcessor $processor) {}
    
    public function checkout(Order $order) {
        $this->processor->process($order->getPayment());
    }
}
```

## Quand appliquer SOLID ?

### Appliquez SOLID quand :

- Vous développez un projet qui évoluera dans le temps
- Plusieurs développeurs travaillent sur le même code
- Vous avez besoin de tester votre code facilement
- Vous voulez réutiliser des composants
- La maintenabilité est importante

### Attention à ne pas :

- **Sur-appliquer** : Un script simple de 50 lignes n'a pas besoin de SOLID
- **Complexifier** : SOLID doit simplifier, pas compliquer
- **Ignorer le contexte** : Parfois, une petite violation est acceptable
- **Oublier le pragmatisme** : L'objectif est un code meilleur, pas un code parfait

## SOLID et les autres pratiques

SOLID s'intègre bien avec :

- **Design Patterns** : SOLID facilite l'application des patterns
- **TDD (Test-Driven Development)** : SOLID rend le code plus testable
- **Clean Code** : SOLID est une partie essentielle du code propre
- **Refactoring** : SOLID guide le refactoring
- **Architecture** : SOLID influence l'architecture globale

## Métriques pour mesurer SOLID

Bien que SOLID soit qualitatif, certains indicateurs peuvent aider :

- **Couplage** : Nombre de dépendances entre classes (devrait être faible)
- **Cohésion** : Liens entre les méthodes d'une classe (devrait être élevée)
- **Complexité cyclomatique** : Complexité des méthodes (devrait être faible)
- **Nombre de responsabilités** : Par classe (devrait être 1 pour SRP)

## Conclusion

SOLID n'est pas une liste de règles à suivre aveuglément, mais un ensemble de principes qui guident vers un code de meilleure qualité. L'objectif est de créer un code :

- **Facile à comprendre** : Chaque classe a un rôle clair
- **Facile à modifier** : Les changements sont localisés
- **Facile à tester** : Les dépendances sont explicites
- **Facile à réutiliser** : Les composants sont découplés
- **Facile à maintenir** : Le code vieillit bien

Dans les chapitres suivants, nous explorerons chaque principe en détail avec des exemples concrets en PHP.
