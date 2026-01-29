# Exemples concrets du pattern Decorator

Ce document présente des exemples complets et réalistes d'application du pattern Decorator, en pseudo-code style Java.

## Exemple 1 : Café (classique)

### Contexte

Un système de commande de café où l'on peut ajouter lait, sucre, chantilly, etc. Les combinaisons sont nombreuses et évolutives.

### Implémentation

```java
// Composant
interface Coffee {
    String getDescription();
    double getCost();
}

// Composant concret
class SimpleCoffee implements Coffee {
    public String getDescription() { return "Simple coffee"; }
    public double getCost() { return 2.0; }
}

// Decorator de base
abstract class CoffeeDecorator implements Coffee {
    protected Coffee coffee;
    public CoffeeDecorator(Coffee coffee) { this.coffee = coffee; }
    public String getDescription() { return coffee.getDescription(); }
    public double getCost() { return coffee.getCost(); }
}

// Decorators concrets
class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) { super(coffee); }
    public String getDescription() { return coffee.getDescription() + ", milk"; }
    public double getCost() { return coffee.getCost() + 0.5; }
}

class SugarDecorator extends CoffeeDecorator {
    public SugarDecorator(Coffee coffee) { super(coffee); }
    public String getDescription() { return coffee.getDescription() + ", sugar"; }
    public double getCost() { return coffee.getCost() + 0.2; }
}

class WhipDecorator extends CoffeeDecorator {
    public WhipDecorator(Coffee coffee) { super(coffee); }
    public String getDescription() { return coffee.getDescription() + ", whip"; }
    public double getCost() { return coffee.getCost() + 0.7; }
}

// Utilisation
Coffee coffee = new SimpleCoffee();
System.out.println(coffee.getDescription() + ": $" + coffee.getCost());  // Simple coffee: $2.0

coffee = new MilkDecorator(coffee);
System.out.println(coffee.getDescription() + ": $" + coffee.getCost());  // Simple coffee, milk: $2.5

coffee = new SugarDecorator(coffee);
System.out.println(coffee.getDescription() + ": $" + coffee.getCost());  // Simple coffee, milk, sugar: $2.7

// Chaînage direct
Coffee fancy = new WhipDecorator(new SugarDecorator(new MilkDecorator(new SimpleCoffee())));
System.out.println(fancy.getDescription() + ": $" + fancy.getCost());   // Simple coffee, milk, sugar, whip: $3.4
```

---

## Exemple 2 : Logger avec timestamp et niveau

### Contexte

Un logger de base qui affiche des messages. On veut pouvoir ajouter un horodatage, un niveau (INFO, ERROR), ou une écriture dans un fichier, sans modifier le logger de base.

### Implémentation

```java
// Composant
interface Logger {
    void log(String message);
}

// Composant concret
class BasicLogger implements Logger {
    public void log(String message) { System.out.println(message); }
}

// Decorator de base
abstract class LoggerDecorator implements Logger {
    protected Logger logger;
    public LoggerDecorator(Logger logger) { this.logger = logger; }
    public void log(String message) { logger.log(message); }
}

// Decorators concrets
class TimestampDecorator extends LoggerDecorator {
    public TimestampDecorator(Logger logger) { super(logger); }
    public void log(String message) {
        String ts = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
        logger.log("[" + ts + "] " + message);
    }
}

class LevelDecorator extends LoggerDecorator {
    private String level;
    public LevelDecorator(Logger logger, String level) { super(logger); this.level = level; }
    public void log(String message) { logger.log("[" + level + "] " + message); }
}

class FileDecorator extends LoggerDecorator {
    private String filename;
    public FileDecorator(Logger logger, String filename) { super(logger); this.filename = filename; }
    public void log(String message) {
        logger.log(message);
        Files.write(Path.of(filename), (message + "\n").getBytes(), StandardOpenOption.APPEND);
    }
}

// Utilisation
Logger logger = new BasicLogger();
logger.log("Hello");  // Hello

logger = new TimestampDecorator(logger);
logger.log("Hello");  // [2026-01-30 14:30:00] Hello

logger = new LevelDecorator(logger, "INFO");
logger.log("Hello");  // [INFO] [2026-01-30 14:30:00] Hello

logger = new FileDecorator(logger, "app.log");
logger.log("Hello");  // Affiche ET écrit dans app.log
```

---

## Exemple 3 : Validation en chaîne (formulaire)

### Contexte

Validation de champs : requis, longueur minimale, format email. Chaque règle est un décorateur ; on enchaîne les validations sans modifier une classe de validation unique.

### Implémentation

```java
// Résultat de validation (équivalent tuple bool + string)
class ValidationResult {
    boolean valid;
    String error;
    ValidationResult(boolean valid, String error) { this.valid = valid; this.error = error; }
}

// Composant
interface Validator {
    ValidationResult validate(String value);
}

// Composant concret : accepte tout
class BaseValidator implements Validator {
    public ValidationResult validate(String value) { return new ValidationResult(true, ""); }
}

// Decorator de base
abstract class ValidatorDecorator implements Validator {
    protected Validator validator;
    public ValidatorDecorator(Validator validator) { this.validator = validator; }
    public ValidationResult validate(String value) { return validator.validate(value); }
}

// Decorators concrets
class RequiredValidator extends ValidatorDecorator {
    public RequiredValidator(Validator validator) { super(validator); }
    public ValidationResult validate(String value) {
        if (value == null || value.trim().isEmpty())
            return new ValidationResult(false, "Champ requis");
        return validator.validate(value);
    }
}

class MinLengthValidator extends ValidatorDecorator {
    private int minLength;
    public MinLengthValidator(Validator validator, int minLength) { super(validator); this.minLength = minLength; }
    public ValidationResult validate(String value) {
        if (value.length() < minLength)
            return new ValidationResult(false, "Minimum " + minLength + " caractères");
        return validator.validate(value);
    }
}

class EmailFormatValidator extends ValidatorDecorator {
    public EmailFormatValidator(Validator validator) { super(validator); }
    public ValidationResult validate(String value) {
        if (!value.contains("@") || !value.split("@")[1].contains("."))
            return new ValidationResult(false, "Format email invalide");
        return validator.validate(value);
    }
}

// Utilisation : email requis, min 5 caractères, format email
Validator emailValidator = new EmailFormatValidator(
    new MinLengthValidator(new RequiredValidator(new BaseValidator()), 5)
);

ValidationResult r = emailValidator.validate("");
// r.valid == false, r.error == "Champ requis"

r = emailValidator.validate("ab");
// r.valid == false, r.error == "Minimum 5 caractères"

r = emailValidator.validate("user@example.com");
// r.valid == true
```

---

## Exemple 4 : Flux de données

### Contexte

Un flux de lecture de données. On veut ajouter compression, chiffrement ou bufferisation par couches, sans modifier la source de base.

### Implémentation

```java
// Composant
interface DataStream {
    String read();
}

// Composant concret
class FileStream implements DataStream {
    private String path;
    public FileStream(String path) { this.path = path; }
    public String read() { return "[File: " + path + "]"; }
}

// Decorator de base
abstract class StreamDecorator implements DataStream {
    protected DataStream stream;
    public StreamDecorator(DataStream stream) { this.stream = stream; }
    public String read() { return stream.read(); }
}

// Decorators concrets (simulés)
class CompressedStream extends StreamDecorator {
    public CompressedStream(DataStream stream) { super(stream); }
    public String read() {
        String data = stream.read();
        return "[Compressed]" + data + "[/Compressed]";
    }
}

class EncryptedStream extends StreamDecorator {
    public EncryptedStream(DataStream stream) { super(stream); }
    public String read() {
        String data = stream.read();
        return "[Encrypted]" + data + "[/Encrypted]";
    }
}

// Utilisation
DataStream stream = new FileStream("data.txt");
stream = new CompressedStream(stream);
stream = new EncryptedStream(stream);
System.out.println(stream.read());
// [Encrypted][Compressed][File: data.txt][/Compressed][/Encrypted]
```

---

## Exemple 5 : Requêtes HTTP (enrichissement des réponses)

### Contexte

Un client HTTP de base qui envoie des requêtes. On veut ajouter du cache, du retry ou du logging sans modifier le client.

### Implémentation

```java
// Composant
interface HttpClient {
    String get(String url);
}

// Composant concret (simulé)
class BasicHttpClient implements HttpClient {
    public String get(String url) { return "Response from " + url; }
}

// Decorator de base
abstract class HttpClientDecorator implements HttpClient {
    protected HttpClient client;
    public HttpClientDecorator(HttpClient client) { this.client = client; }
    public String get(String url) { return client.get(url); }
}

// Decorator : cache simple
class CachingDecorator extends HttpClientDecorator {
    private Map<String, String> cache = new HashMap<>();
    public CachingDecorator(HttpClient client) { super(client); }
    public String get(String url) {
        if (cache.containsKey(url))
            return cache.get(url) + " (cached)";
        String result = client.get(url);
        cache.put(url, result);
        return result;
    }
}

// Decorator : log des appels
class LoggingDecorator extends HttpClientDecorator {
    public LoggingDecorator(HttpClient client) { super(client); }
    public String get(String url) {
        System.out.println("GET " + url);
        String result = client.get(url);
        System.out.println("  -> " + result);
        return result;
    }
}

// Utilisation
HttpClient client = new BasicHttpClient();
client = new LoggingDecorator(client);
client = new CachingDecorator(client);

System.out.println(client.get("https://api.example.com/users"));  // GET + Response (puis cached)
System.out.println(client.get("https://api.example.com/users"));  // GET + Response (cached)
```

---

## Synthèse des cas d'usage

| Domaine        | Composant de base | Decorators typiques                    |
|----------------|-------------------|----------------------------------------|
| Boissons       | Café simple       | Lait, sucre, chantilly                 |
| Logging        | Logger console    | Timestamp, niveau, fichier             |
| Validation     | Validateur vide   | Requis, longueur min, format email      |
| I/O            | Flux fichier      | Compression, chiffrement, buffer       |
| HTTP           | Client HTTP       | Cache, retry, logging, authentification |

Dans tous les cas, on **étend le comportement par composition** (enveloppement) et on garde **la même interface** pour le client.

---

**Prochaine étape** : [Mise en pratique](./04-mise-en-pratique.md) — quand utiliser le Decorator, alternatives, pièges à éviter.
