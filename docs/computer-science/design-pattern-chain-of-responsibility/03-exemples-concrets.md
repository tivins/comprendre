# Exemples concrets de la Chaîne de responsabilité

Ce document présente des exemples complets et réalistes d'application du pattern Chaîne de responsabilité, en pseudo-code style Java.

## Exemple 1 : Validation de formulaire (email)

### Contexte

Validation d'un champ email : champ requis, longueur minimale, format email, pas d'espaces. Chaque règle est un handler ; la requête (la valeur à valider) circule jusqu'à la première erreur ou jusqu'à la fin de la chaîne.

### Implémentation

```java
// Résultat de validation
class ValidationResult {
    boolean valid;
    String error;
    ValidationResult(boolean valid, String error) { this.valid = valid; this.error = error; }
}

// Handler abstrait
abstract class Validator {
    protected Validator next;

    public Validator setNext(Validator next) {
        this.next = next;
        return next;
    }

    public ValidationResult validate(String value) {
        ValidationResult result = doValidate(value);
        if (!result.valid) {
            return result;
        }
        if (next != null) {
            return next.validate(value);
        }
        return result;
    }

    protected abstract ValidationResult doValidate(String value);
}

// Handlers concrets
class RequiredValidator extends Validator {
    protected ValidationResult doValidate(String value) {
        if (value == null || value.trim().isEmpty()) {
            return new ValidationResult(false, "Champ requis");
        }
        return new ValidationResult(true, "");
    }
}

class MinLengthValidator extends Validator {
    private int minLength;
    public MinLengthValidator(int minLength) { this.minLength = minLength; }
    protected ValidationResult doValidate(String value) {
        if (value.length() < minLength) {
            return new ValidationResult(false, "Minimum " + minLength + " caractères");
        }
        return new ValidationResult(true, "");
    }
}

class EmailFormatValidator extends Validator {
    protected ValidationResult doValidate(String value) {
        if (!value.contains("@") || !value.split("@")[1].contains(".")) {
            return new ValidationResult(false, "Format email invalide");
        }
        return new ValidationResult(true, "");
    }
}

class NoSpacesValidator extends Validator {
    protected ValidationResult doValidate(String value) {
        if (value.contains(" ")) {
            return new ValidationResult(false, "L'email ne doit pas contenir d'espaces");
        }
        return new ValidationResult(true, "");
    }
}

// Utilisation
Validator emailValidator = new RequiredValidator();
emailValidator.setNext(new MinLengthValidator(5))
             .setNext(new EmailFormatValidator())
             .setNext(new NoSpacesValidator());

ValidationResult r = emailValidator.validate("");
// r.valid == false, r.error == "Champ requis"

r = emailValidator.validate("ab");
// r.valid == false, r.error == "Minimum 5 caractères"

r = emailValidator.validate("user @example.com");
// r.valid == false, r.error == "L'email ne doit pas contenir d'espaces"

r = emailValidator.validate("user@example.com");
// r.valid == true
```

---

## Exemple 2 : Gestion des erreurs HTTP

### Contexte

Une application doit réagir différemment selon le code d'erreur HTTP (404, 401, 500, etc.). Chaque handler gère un type d'erreur ; la requête (code + message) circule jusqu'à ce qu'un handler la prenne en charge (ou jusqu'au handler par défaut).

### Implémentation

```java
// Contexte de l'erreur
class ErrorContext {
    int code;
    String message;
    public ErrorContext(int code, String message) { this.code = code; this.message = message; }
}

// Handler abstrait
abstract class ErrorHandler {
    protected ErrorHandler next;

    public ErrorHandler setNext(ErrorHandler next) {
        this.next = next;
        return next;
    }

    public boolean handle(ErrorContext ctx) {
        if (canHandle(ctx)) {
            doHandle(ctx);
            return true;
        }
        if (next != null) {
            return next.handle(ctx);
        }
        return false;
    }

    protected abstract boolean canHandle(ErrorContext ctx);
    protected abstract void doHandle(ErrorContext ctx);
}

// Handlers concrets
class NotFoundHandler extends ErrorHandler {
    protected boolean canHandle(ErrorContext ctx) { return ctx.code == 404; }
    protected void doHandle(ErrorContext ctx) {
        System.out.println("404 : Page non trouvée - " + ctx.message);
    }
}

class UnauthorizedHandler extends ErrorHandler {
    protected boolean canHandle(ErrorContext ctx) { return ctx.code == 401; }
    protected void doHandle(ErrorContext ctx) {
        System.out.println("401 : Redirection vers la page de connexion - " + ctx.message);
    }
}

class ServerErrorHandler extends ErrorHandler {
    protected boolean canHandle(ErrorContext ctx) { return ctx.code >= 500 && ctx.code < 600; }
    protected void doHandle(ErrorContext ctx) {
        System.out.println("500 : Erreur serveur - " + ctx.message);
    }
}

class DefaultErrorHandler extends ErrorHandler {
    protected boolean canHandle(ErrorContext ctx) { return true; }
    protected void doHandle(ErrorContext ctx) {
        System.out.println("Erreur inconnue " + ctx.code + " : " + ctx.message);
    }
}

// Utilisation
ErrorHandler chain = new NotFoundHandler();
chain.setNext(new UnauthorizedHandler())
     .setNext(new ServerErrorHandler())
     .setNext(new DefaultErrorHandler());

chain.handle(new ErrorContext(404, "Ressource introuvable"));
chain.handle(new ErrorContext(401, "Authentification requise"));
chain.handle(new ErrorContext(500, "Erreur interne du serveur"));
chain.handle(new ErrorContext(403, "Accès refusé"));  // Pris en charge par DefaultErrorHandler
```

---

## Exemple 3 : Niveaux d'approbation (demande de congés)

### Contexte

Une demande de congés doit être approuvée selon le nombre de jours : jusqu'à 3 jours → manager direct ; jusqu'à 10 jours → directeur ; au-delà → direction générale. Chaque handler représente un niveau ; la requête (demande) circule jusqu'à ce qu'un niveau l'approuve (ou la rejette).

### Implémentation

```java
// Demande
class LeaveRequest {
    String employee;
    int days;
    public LeaveRequest(String employee, int days) { this.employee = employee; this.days = days; }
}

// Handler abstrait
abstract class Approver {
    protected Approver next;
    protected String role;

    public Approver setNext(Approver next) {
        this.next = next;
        return next;
    }

    public void process(LeaveRequest request) {
        if (canApprove(request)) {
            System.out.println(role + " approuve la demande de " + request.days + " jours pour " + request.employee);
            return;
        }
        if (next != null) {
            next.process(request);
        } else {
            System.out.println("Aucun approbateur ne peut traiter cette demande.");
        }
    }

    protected abstract boolean canApprove(LeaveRequest request);
}

class ManagerApprover extends Approver {
    public ManagerApprover() { this.role = "Manager"; }
    protected boolean canApprove(LeaveRequest request) { return request.days <= 3; }
}

class DirectorApprover extends Approver {
    public DirectorApprover() { this.role = "Directeur"; }
    protected boolean canApprove(LeaveRequest request) { return request.days <= 10; }
}

class GeneralDirectorApprover extends Approver {
    public GeneralDirectorApprover() { this.role = "Direction générale"; }
    protected boolean canApprove(LeaveRequest request) { return true; }
}

// Utilisation
Approver chain = new ManagerApprover();
chain.setNext(new DirectorApprover()).setNext(new GeneralDirectorApprover());

chain.process(new LeaveRequest("Alice", 2));   // Manager approuve
chain.process(new LeaveRequest("Bob", 7));    // Directeur approuve
chain.process(new LeaveRequest("Charlie", 15)); // Direction générale approuve
```

---

## Exemple 4 : Pipeline de traitement (nourriture pour animaux)

### Contexte

Une requête représente un type de nourriture. Plusieurs "handlers" (animaux) peuvent la manger ; la requête circule jusqu'à ce qu'un animal la prenne (ex. singe → banane, écureuil → noix, chien → boulette). Classique du GoF.

### Implémentation

```java
// Handler abstrait
abstract class AnimalHandler {
    protected AnimalHandler next;

    public AnimalHandler setNext(AnimalHandler next) {
        this.next = next;
        return next;
    }

    public String handle(String food) {
        if (canEat(food)) {
            return getResponse(food);
        }
        if (next != null) {
            return next.handle(food);
        }
        return null;  // Personne n'a mangé
    }

    protected abstract boolean canEat(String food);
    protected abstract String getResponse(String food);
}

class MonkeyHandler extends AnimalHandler {
    protected boolean canEat(String food) { return "Banana".equals(food); }
    protected String getResponse(String food) { return "Singe : je mange la " + food; }
}

class SquirrelHandler extends AnimalHandler {
    protected boolean canEat(String food) { return "Nut".equals(food); }
    protected String getResponse(String food) { return "Écureuil : je mange la " + food; }
}

class DogHandler extends AnimalHandler {
    protected boolean canEat(String food) { return "MeatBall".equals(food); }
    protected String getResponse(String food) { return "Chien : je mange la " + food; }
}

// Utilisation
AnimalHandler chain = new MonkeyHandler();
chain.setNext(new SquirrelHandler()).setNext(new DogHandler());

String result = chain.handle("Nut");       // "Écureuil : je mange la Nut"
result = chain.handle("Banana");            // "Singe : je mange la Banana"
result = chain.handle("Cup of coffee");     // null (personne ne mange)
```

---

## Synthèse des cas d'usage

| Domaine        | Requête / Contexte     | Handlers typiques                    | Comportement                    |
|----------------|------------------------|--------------------------------------|---------------------------------|
| Validation     | Valeur à valider       | Requis, longueur min, format, espaces | Premier échec arrête la chaîne  |
| Erreurs HTTP   | Code + message         | 404, 401, 500, défaut                | Un seul handler traite          |
| Approbation    | Demande (montant, jours) | Manager, directeur, direction       | Premier niveau compétent traite |
| Pipeline       | Type de requête        | Singe, écureuil, chien               | Premier handler concerné traite |

Dans tous les cas : **découplage** entre l'émetteur et les traitants, **requête qui circule** jusqu'à prise en charge, **ordre de la chaîne** configurable.

---

**Prochaine étape** : [Mise en pratique](./04-mise-en-pratique.md) — quand utiliser la Chaîne de responsabilité, alternatives, pièges à éviter.
