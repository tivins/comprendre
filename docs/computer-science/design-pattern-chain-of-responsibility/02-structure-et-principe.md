# Structure et principe de la Chaîne de responsabilité

## Rôles des composants

Le pattern Chaîne de responsabilité met en jeu trois types d'éléments : le **Handler** (abstrait), les **Handlers concrets** et éventuellement un objet **Requête** (ou un contexte) qui circule le long de la chaîne.

### 1. Handler (abstrait)

**Rôle** : Définir l'interface commune à tous les maillons de la chaîne. Déclarer la méthode qui traite la requête (ex. `handle(Request request)`). Gérer la référence vers le **successeur** (next handler) et la logique de passage au successeur si le handler ne traite pas la requête.

**En code** : Une classe abstraite (ou une interface avec une implémentation de base) qui possède une référence vers le prochain handler et une méthode pour l'enregistrer (`setNext`). La méthode `handle` appelle éventuellement le successeur.

```java
abstract class Handler {
    protected Handler next;

    public Handler setNext(Handler next) {
        this.next = next;
        return next;  // Permet le chaînage fluide : chain.setNext(A).setNext(B)
    }

    public abstract boolean handle(Request request);
}
```

### 2. Handlers concrets (Concrete Handlers)

**Rôle** : Chaque handler concret décide s'il peut traiter la requête. S'il peut, il la traite et peut soit retourner un résultat, soit arrêter la propagation. S'il ne peut pas, il transmet la requête au successeur (s'il existe).

**En code** : Sous-classes du Handler qui implémentent la logique métier (ex. "suis-je concerné par cette requête ?") et appellent `next.handle(request)` si elles ne traitent pas.

```java
class ValidationHandler extends Handler {
    public boolean handle(Request request) {
        if (request.getType().equals("VALIDATION")) {
            // Traiter la validation
            return true;  // Requête traitée, on arrête
        }
        if (next != null) {
            return next.handle(request);
        }
        return false;  // Personne n'a traité
    }
}

class Error404Handler extends Handler {
    public boolean handle(Request request) {
        if (request.getCode() == 404) {
            // Gérer l'erreur 404
            return true;
        }
        if (next != null) {
            return next.handle(request);
        }
        return false;
    }
}
```

### 3. Requête / Contexte

**Rôle** : Objet (ou paramètres) qui circule le long de la chaîne. Il contient les informations nécessaires pour que chaque handler décide s'il traite et comment il traite.

**En code** : Une classe ou un DTO (Data Transfer Object) avec les champs utiles (type, code, message, données métier, etc.).

```java
class Request {
    private String type;
    private int code;
    private String message;
    // getters, setters, constructeur
}
```

## Schéma des relations

```
        ┌─────────────────┐
        │     Client      │
        └────────┬────────┘
                 │ handle(request)
                 ▼
        ┌─────────────────┐
        │    Handler      │  (abstrait)
        │  - next         │
        │  + setNext()    │
        │  + handle()     │
        └────────┬────────┘
                 │
        ┌────────┴────────────────────────┐
        │                                 │
        ▼                                 ▼
┌───────────────┐                 ┌───────────────┐
│  Concrete     │  ──next──►      │  Concrete     │  ──next──►  ...
│  Handler A    │                 │  Handler B    │
│  + handle()   │                 │  + handle()   │
└───────────────┘                 └───────────────┘
```

- Le **Client** ne connaît que le **premier Handler**.
- Chaque **Concrete Handler** peut avoir un **successeur** (next).
- La **Requête** est passée de handler en handler jusqu'à ce qu'un maillon la traite (ou jusqu'à la fin de la chaîne).

## Principe de passage de la requête

Un handler typique :

1. **Reçoit** la requête (ex. `handle(request)`).
2. **Décide** s'il peut la traiter (condition sur le type, le code, les données, etc.).
3. **Si oui** : il effectue le traitement et peut retourner (arrêt de la propagation) ou continuer selon le besoin.
4. **Si non** : il appelle `next.handle(request)` si `next != null`, sinon il peut retourner une valeur par défaut (ex. "non traité").

Exemple avec arrêt dès qu'un handler traite :

```java
public boolean handle(Request request) {
    if (canHandle(request)) {
        doHandle(request);
        return true;
    }
    if (next != null) {
        return next.handle(request);
    }
    return false;
}
```

Variante où **tous** les handlers concernés traitent (ex. validation en chaîne : chaque validateur applique sa règle et passe au suivant) :

```java
public ValidationResult handle(String value) {
    ValidationResult result = validate(value);  // Ma règle
    if (!result.isValid()) {
        return result;
    }
    if (next != null) {
        return next.handle(value);
    }
    return result;
}
```

## Construction de la chaîne

La chaîne est construite en enregistrant le successeur de chaque maillon. Un `setNext` qui retourne le handler passé en paramètre permet un chaînage fluide :

```java
Handler chain = new HandlerA();
chain.setNext(new HandlerB()).setNext(new HandlerC()).setNext(new HandlerD());

// Le client n'appelle que le premier maillon
chain.handle(request);
```

L'ordre des maillons a du sens : il définit la **priorité** ou l'**ordre de traitement** (ex. validation "requis" avant "format email", erreur 404 avant erreur 500 dans un fallback).

## Qui traite la requête ?

- **Un seul handler** : souvent, dès qu'un handler traite la requête, la propagation s'arrête (return après traitement).
- **Plusieurs handlers** : dans certains cas (ex. pipeline de validation), chaque handler fait son travail et passe au suivant ; la requête traverse toute la chaîne ou s'arrête à la première erreur selon le design.

Le choix dépend du domaine : gestion d'erreurs HTTP → en général un seul handler par requête ; validation de formulaire → souvent tous les validateurs s'enchaînent jusqu'à la première erreur.

## En résumé

| Élément            | Rôle principal                                      |
|--------------------|-----------------------------------------------------|
| **Handler**        | Interface / base commune ; référence vers next ; méthode handle |
| **Concrete Handler** | Décide s'il traite la requête ; sinon appelle next.handle() |
| **Requête / Contexte** | Données qui circulent le long de la chaîne          |
| **Client**         | Construit la chaîne (setNext) et appelle le premier handler |

Le cœur du pattern : **chaîne de successeurs + passage de la requête jusqu'à prise en charge** (ou jusqu'à la fin). Le client reste découplé de la liste des traitants.

---

**Prochaine étape** : [Exemples concrets](./03-exemples-concrets.md) (validation, erreurs HTTP, pipeline, niveaux d'approbation).
