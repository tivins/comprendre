# Interface Segregation Principle (ISP)

## Définition

**Les clients ne devraient pas être forcés de dépendre d'interfaces qu'ils n'utilisent pas.**

Le principe de ségrégation des interfaces (ISP) stipule qu'il vaut mieux avoir plusieurs interfaces spécifiques qu'une seule interface générale. Une classe ne devrait pas être obligée d'implémenter des méthodes qu'elle n'utilise pas.

## Comprendre ISP

### Concept de base

Si vous avez une interface avec 10 méthodes, mais qu'une classe n'en utilise que 3, cette classe est forcée d'implémenter les 7 autres méthodes (même si c'est vide ou avec une exception). C'est une violation de l'ISP.

### Analogie

Imaginez une télécommande universelle avec 100 boutons :
- Pour allumer la TV, vous n'avez besoin que de quelques boutons
- Mais vous êtes forcé d'avoir tous les boutons, même ceux que vous n'utilisez jamais
- Mieux vaut avoir plusieurs télécommandes spécialisées (TV, radio, climatisation)

## Pourquoi ISP est important ?

### Problèmes sans ISP

```php
// À éviter : Interface trop large
interface Worker {
    public function work(): void;
    public function eat(): void;
    public function sleep(): void;
}

class Human implements Worker {
    public function work(): void {
        echo "Human working\n";
    }
    
    public function eat(): void {
        echo "Human eating\n";
    }
    
    public function sleep(): void {
        echo "Human sleeping\n";
    }
}

class Robot implements Worker {
    public function work(): void {
        echo "Robot working\n";
    }
    
    public function eat(): void {
        // Robot ne mange pas ! Obligé d'implémenter une méthode inutile
        throw new Exception("Robots don't eat");
    }
    
    public function sleep(): void {
        // Robot ne dort pas ! Obligé d'implémenter une méthode inutile
        throw new Exception("Robots don't sleep");
    }
}
```

**Problèmes** :
- `Robot` est forcé d'implémenter des méthodes qu'il n'utilise pas
- Code inutile (méthodes vides ou avec exceptions)
- Violation du principe de responsabilité unique
- Couplage inutile avec des fonctionnalités non utilisées

## Solution : Interfaces séparées

```php
// Bon : Interfaces spécifiques
interface Workable {
    public function work(): void;
}

interface Eatable {
    public function eat(): void;
}

interface Sleepable {
    public function sleep(): void;
}

class Human implements Workable, Eatable, Sleepable {
    public function work(): void {
        echo "Human working\n";
    }
    
    public function eat(): void {
        echo "Human eating\n";
    }
    
    public function sleep(): void {
        echo "Human sleeping\n";
    }
}

class Robot implements Workable {
    public function work(): void {
        echo "Robot working\n";
    }
    // Pas besoin d'implémenter eat() ou sleep()
}

class Cat implements Eatable, Sleepable {
    public function eat(): void {
        echo "Cat eating\n";
    }
    
    public function sleep(): void {
        echo "Cat sleeping\n";
    }
    // Pas besoin d'implémenter work()
}
```

## Exemples détaillés

### Exemple 1 : Gestion de documents

```php
// À éviter : Interface trop large
interface Document {
    public function open(): void;
    public function save(): void;
    public function print(): void;
    public function email(): void;
    public function fax(): void;
}

class TextDocument implements Document {
    public function open(): void { }
    public function save(): void { }
    public function print(): void { }
    public function email(): void { }
    public function fax(): void {
        // Les documents texte ne peuvent pas être faxés
        throw new Exception("Text documents cannot be faxed");
    }
}

class ReadOnlyDocument implements Document {
    public function open(): void { }
    public function save(): void {
        // Document en lecture seule
        throw new Exception("Cannot save read-only document");
    }
    public function print(): void { }
    public function email(): void { }
    public function fax(): void { }
}

// Bon : Interfaces séparées
interface Openable {
    public function open(): void;
}

interface Savable {
    public function save(): void;
}

interface Printable {
    public function print(): void;
}

interface Emailable {
    public function email(): void;
}

interface Faxable {
    public function fax(): void;
}

class TextDocument implements Openable, Savable, Printable, Emailable {
    public function open(): void { }
    public function save(): void { }
    public function print(): void { }
    public function email(): void { }
    // Pas besoin d'implémenter Faxable
}

class ReadOnlyDocument implements Openable, Printable {
    public function open(): void { }
    public function print(): void { }
    // Pas besoin d'implémenter Savable, Emailable, Faxable
}

class FaxDocument implements Openable, Faxable {
    public function open(): void { }
    public function fax(): void { }
}
```

### Exemple 2 : Système de paiement

```php
// À éviter : Interface unique pour tous les types de paiement
interface PaymentMethod {
    public function authorize(float $amount): bool;
    public function capture(float $amount): bool;
    public function refund(float $amount): bool;
    public function void(float $amount): bool;
    public function getBalance(): float;
}

class CreditCardPayment implements PaymentMethod {
    public function authorize(float $amount): bool { return true; }
    public function capture(float $amount): bool { return true; }
    public function refund(float $amount): bool { return true; }
    public function void(float $amount): bool { return true; }
    public function getBalance(): float {
        // Les cartes de crédit n'ont pas de "balance" comme un compte
        throw new Exception("Credit cards don't have balance");
    }
}

class BankAccountPayment implements PaymentMethod {
    public function authorize(float $amount): bool {
        // Les comptes bancaires n'ont pas besoin d'autorisation
        throw new Exception("Bank accounts don't need authorization");
    }
    public function capture(float $amount): bool { return true; }
    public function refund(float $amount): bool { return true; }
    public function void(float $amount): bool {
        // Pas applicable aux comptes bancaires
        throw new Exception("Cannot void bank account payment");
    }
    public function getBalance(): float { return 1000.0; }
}

// Bon : Interfaces séparées par capacité
interface Authorizable {
    public function authorize(float $amount): bool;
}

interface Capturable {
    public function capture(float $amount): bool;
}

interface Refundable {
    public function refund(float $amount): bool;
}

interface Voidable {
    public function void(float $amount): bool;
}

interface BalanceCheckable {
    public function getBalance(): float;
}

class CreditCardPayment implements Authorizable, Capturable, Refundable, Voidable {
    public function authorize(float $amount): bool { return true; }
    public function capture(float $amount): bool { return true; }
    public function refund(float $amount): bool { return true; }
    public function void(float $amount): bool { return true; }
}

class BankAccountPayment implements Capturable, Refundable, BalanceCheckable {
    public function capture(float $amount): bool { return true; }
    public function refund(float $amount): bool { return true; }
    public function getBalance(): float { return 1000.0; }
}

class CashPayment implements Capturable {
    public function capture(float $amount): bool { return true; }
    // Pas d'autorisation, remboursement, void, ou balance
}
```

### Exemple 3 : Système de configuration

```php
// À éviter : Interface unique pour toutes les sources de configuration
interface ConfigSource {
    public function get(string $key): mixed;
    public function set(string $key, mixed $value): void;
    public function delete(string $key): void;
    public function exists(string $key): bool;
    public function getAll(): array;
    public function save(): void;
}

class EnvironmentConfig implements ConfigSource {
    public function get(string $key): mixed {
        return $_ENV[$key] ?? null;
    }
    
    public function set(string $key, mixed $value): void {
        // On ne peut pas modifier les variables d'environnement
        throw new Exception("Cannot set environment variables");
    }
    
    public function delete(string $key): void {
        // On ne peut pas supprimer les variables d'environnement
        throw new Exception("Cannot delete environment variables");
    }
    
    public function exists(string $key): bool {
        return isset($_ENV[$key]);
    }
    
    public function getAll(): array {
        return $_ENV;
    }
    
    public function save(): void {
        // Pas de sauvegarde pour les variables d'environnement
        throw new Exception("Cannot save environment variables");
    }
}

// Bon : Interfaces séparées
interface ReadableConfig {
    public function get(string $key): mixed;
    public function exists(string $key): bool;
    public function getAll(): array;
}

interface WritableConfig {
    public function set(string $key, mixed $value): void;
    public function delete(string $key): void;
}

interface PersistableConfig {
    public function save(): void;
}

class EnvironmentConfig implements ReadableConfig {
    public function get(string $key): mixed {
        return $_ENV[$key] ?? null;
    }
    
    public function exists(string $key): bool {
        return isset($_ENV[$key]);
    }
    
    public function getAll(): array {
        return $_ENV;
    }
}

class FileConfig implements ReadableConfig, WritableConfig, PersistableConfig {
    private array $config = [];
    
    public function get(string $key): mixed {
        return $this->config[$key] ?? null;
    }
    
    public function exists(string $key): bool {
        return isset($this->config[$key]);
    }
    
    public function getAll(): array {
        return $this->config;
    }
    
    public function set(string $key, mixed $value): void {
        $this->config[$key] = $value;
    }
    
    public function delete(string $key): void {
        unset($this->config[$key]);
    }
    
    public function save(): void {
        file_put_contents('config.json', json_encode($this->config));
    }
}
```

### Exemple 4 : Système de notifications

```php
// À éviter : Interface unique pour tous les canaux
interface NotificationChannel {
    public function send(string $message): void;
    public function sendWithAttachment(string $message, string $file): void;
    public function sendBulk(array $messages): void;
    public function getDeliveryStatus(string $messageId): string;
}

class EmailChannel implements NotificationChannel {
    public function send(string $message): void { }
    public function sendWithAttachment(string $message, string $file): void { }
    public function sendBulk(array $messages): void { }
    public function getDeliveryStatus(string $messageId): string {
        // Les emails peuvent avoir un statut
        return 'delivered';
    }
}

class SMSSChannel implements NotificationChannel {
    public function send(string $message): void { }
    public function sendWithAttachment(string $message, string $file): void {
        // Les SMS ne supportent pas les pièces jointes
        throw new Exception("SMS does not support attachments");
    }
    public function sendBulk(array $messages): void { }
    public function getDeliveryStatus(string $messageId): string {
        // Les SMS peuvent avoir un statut
        return 'sent';
    }
}

class PushNotificationChannel implements NotificationChannel {
    public function send(string $message): void { }
    public function sendWithAttachment(string $message, string $file): void {
        // Les notifications push peuvent avoir des images mais pas de fichiers
        throw new Exception("Push notifications don't support file attachments");
    }
    public function sendBulk(array $messages): void {
        // Les notifications push ne supportent pas le bulk
        throw new Exception("Push notifications don't support bulk sending");
    }
    public function getDeliveryStatus(string $messageId): string {
        // Pas de statut de livraison pour les push
        throw new Exception("Push notifications don't have delivery status");
    }
}

// Bon : Interfaces séparées
interface Sendable {
    public function send(string $message): void;
}

interface AttachmentSendable {
    public function sendWithAttachment(string $message, string $file): void;
}

interface BulkSendable {
    public function sendBulk(array $messages): void;
}

interface StatusTrackable {
    public function getDeliveryStatus(string $messageId): string;
}

class EmailChannel implements Sendable, AttachmentSendable, BulkSendable, StatusTrackable {
    public function send(string $message): void { }
    public function sendWithAttachment(string $message, string $file): void { }
    public function sendBulk(array $messages): void { }
    public function getDeliveryStatus(string $messageId): string {
        return 'delivered';
    }
}

class SMSSChannel implements Sendable, BulkSendable, StatusTrackable {
    public function send(string $message): void { }
    public function sendBulk(array $messages): void { }
    public function getDeliveryStatus(string $messageId): string {
        return 'sent';
    }
}

class PushNotificationChannel implements Sendable {
    public function send(string $message): void { }
    // Pas besoin d'implémenter les autres interfaces
}
```

## Comment identifier les violations de l'ISP

### Signes de violation

1. **Méthodes vides** : Si une classe implémente des méthodes vides
2. **Exceptions dans les implémentations** : Si une classe lance des exceptions dans des méthodes d'interface
3. **Commentaires "Not implemented"** : Si vous devez mettre des commentaires expliquant pourquoi une méthode n'est pas implémentée
4. **Clients qui n'utilisent qu'une partie** : Si les clients n'utilisent qu'un sous-ensemble des méthodes

### Test

```php
// Si vous pouvez créer une interface plus petite qui satisfait un client,
// l'interface originale viole probablement l'ISP
interface MinimalInterface {
    // Seulement les méthodes vraiment utilisées
}
```

## Avantages de l'ISP

1. **Découplage** : Les classes ne dépendent que de ce dont elles ont besoin
2. **Flexibilité** : Facile d'ajouter de nouvelles implémentations
3. **Maintenabilité** : Modifications localisées, moins d'effets de bord
4. **Clarté** : Interfaces claires montrent exactement ce qui est nécessaire
5. **Réutilisabilité** : Interfaces spécifiques peuvent être réutilisées

## Pièges à éviter

### 1. Sur-ségrégation

```php
// À éviter : Trop de petites interfaces
interface GetName {
    public function getName(): string;
}
interface GetEmail {
    public function getEmail(): string;
}
interface GetAge {
    public function getAge(): int;
}

// Bon : Une interface cohérente pour les données utilisateur
interface UserData {
    public function getName(): string;
    public function getEmail(): string;
    public function getAge(): int;
}
```

### 2. Ignorer la cohésion

Les méthodes qui vont ensemble devraient être dans la même interface.

### 3. Créer des interfaces "au cas où"

Ne créez pas d'interfaces pour des fonctionnalités futures hypothétiques.

## Comment appliquer l'ISP

1. **Identifier** les groupes de méthodes utilisées ensemble
2. **Séparer** les interfaces par responsabilité ou capacité
3. **Implémenter** seulement les interfaces nécessaires
4. **Refactoriser** progressivement les interfaces existantes
5. **Tester** que les clients peuvent utiliser seulement ce dont ils ont besoin

## ISP et les autres principes SOLID

- **SRP** : ISP aide à respecter SRP en séparant les responsabilités dans des interfaces
- **OCP** : ISP facilite l'extension en permettant d'ajouter de nouvelles implémentations
- **LSP** : ISP aide à respecter LSP en créant des contrats plus précis
- **DIP** : ISP améliore DIP en créant des abstractions plus spécifiques

## Conclusion

Le principe de ségrégation des interfaces garantit que les classes ne sont pas forcées d'implémenter des méthodes qu'elles n'utilisent pas. En créant des interfaces spécifiques et ciblées, vous créez un code plus flexible, maintenable et découplé. **Plusieurs interfaces spécifiques valent mieux qu'une seule interface générale**.
