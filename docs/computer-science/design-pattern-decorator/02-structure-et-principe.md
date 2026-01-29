# Structure et principe du Decorator

## Rôles des composants

Le pattern Decorator met en jeu quatre types d'éléments : le **Composant** (abstrait), le **Composant concret**, le **Decorator** (abstrait) et les **Decorators concrets**.

### 1. Composant (Component)

**Rôle** : Définir l'interface commune à l'objet de base et à tous les décorateurs. C'est l'abstraction vers laquelle le client programme.

**En code** : Une interface (ou classe abstraite) avec les méthodes que le composant et les décorateurs doivent exposer (ex. `getCost()`, `getDescription()`).

```java
interface Coffee {
    String getDescription();
    double getCost();
}
```

### 2. Composant concret (Concrete Component)

**Rôle** : L'objet de base, sans décoration. Il implémente le Composant et fournit le comportement minimal.

**En code** : Une classe concrète qui implémente le Composant.

```java
class SimpleCoffee implements Coffee {
    public String getDescription() { return "Simple coffee"; }
    public double getCost() { return 2.0; }
}
```

### 3. Decorator (Decorator de base)

**Rôle** : Garder une référence vers un Composant (le "composant décoré") et implémenter la même interface. Il délègue les appels au composant qu'il enveloppe ; les sous-classes ajoutent du comportement avant ou après la délégation.

**En code** : Une classe qui implémente le Composant, reçoit un Composant en constructeur, et redéfinit les méthodes en appelant celles du composant décoré (éventuellement en les enrichissant).

```java
abstract class CoffeeDecorator implements Coffee {
    protected Coffee coffee;

    public CoffeeDecorator(Coffee coffee) {
        this.coffee = coffee;
    }

    public String getDescription() { return coffee.getDescription(); }
    public double getCost() { return coffee.getCost(); }
}
```

### 4. Decorators concrets (Concrete Decorators)

**Rôle** : Chaque décorateur concret ajoute une responsabilité précise. Il appelle en général le comportement du décorateur/composant qu'il enveloppe, puis ajoute le sien (avant ou après).

**En code** : Sous-classes du Decorator qui surchargent les méthodes pour ajouter coût, description, log, etc.

```java
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
```

## Schéma des relations

```
        ┌─────────────────┐
        │   Component     │  (interface / classe abstraite)
        │  + operation()  │
        └────────┬────────┘
                 │
        ┌────────┴────────┐
        │                 │
        ▼                 ▼
┌───────────────┐  ┌─────────────────┐
│   Concrete    │  │    Decorator    │  (référence vers Component)
│   Component   │  │  - component    │
│  + operation()│  │  + operation()  │  → délègue à component
└───────────────┘  └────────┬────────┘
                            │
                   ┌────────┴────────┐
                   ▼                 ▼
           ┌───────────────┐  ┌───────────────┐
           │  Concrete     │  │  Concrete     │
           │  Decorator A  │  │  Decorator B  │
           │  + operation()│  │  + operation()│
           └───────────────┘  └───────────────┘
```

- Le **Client** ne dépend que du **Component**.
- Le **Concrete Component** et les **Concrete Decorators** sont tous utilisables à la place du Component (substitution).
- Chaque **Decorator** contient un **Component** (composition) et délègue vers lui.

## Principe de délégation

Un décorateur ne remplace pas le composant : il **enveloppe** un composant (qui peut lui-même être un autre décorateur) et :

1. **Reçoit** l'appel du client (ex. `get_cost()`).
2. **Optionnel** : fait un travail avant (ex. log, validation).
3. **Délègue** au composant décoré (ex. `coffee.getCost()`).
4. **Optionnel** : fait un travail après (ex. ajouter un coût, formater la sortie).
5. **Retourne** le résultat au client.

Exemple avec ajout avant et après :

```java
class TimestampDecorator extends LoggerDecorator {
    public TimestampDecorator(Logger logger) { super(logger); }
    public void log(String message) {
        String timestamp = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));  // Avant
        logger.log("[" + timestamp + "] " + message);  // Délégation
    }
}
```

## Chaînage des décorateurs

On peut enchaîner plusieurs décorateurs : chaque décorateur enveloppe le précédent. L'ordre d'appel des méthodes est alors "de l'extérieur vers l'intérieur" (le dernier décorateur appelé est le premier à s'exécuter jusqu'à la délégation).

```java
// Chaîne : Whip → Sugar → Milk → SimpleCoffee
Coffee coffee = new WhipDecorator(new SugarDecorator(new MilkDecorator(new SimpleCoffee())));

// getCost() : WhipDecorator appelle SugarDecorator, qui appelle MilkDecorator,
// qui appelle SimpleCoffee ; les coûts s'ajoutent au retour.
System.out.println(coffee.getCost());  // 2.0 + 0.5 + 0.2 + 0.7 = 3.4
```

L'ordre d'enveloppement peut changer le résultat si les décorateurs modifient les données (ex. format de message, ordre de validation). À documenter ou à fixer par convention selon le cas.

## Même interface pour le client

Le client travaille uniquement avec le type **Component**. Il ne sait pas si l'objet est un SimpleCoffee ou une pile de décorateurs. C'est le **principe de substitution** : tout décorateur et tout composant concret sont substituables au composant abstrait.

```java
void printOrder(Coffee coffee) {
    System.out.println(coffee.getDescription() + ": $" + String.format("%.2f", coffee.getCost()));
}

// Tous ces appels sont valides
printOrder(new SimpleCoffee());
printOrder(new MilkDecorator(new SimpleCoffee()));
printOrder(new WhipDecorator(new SugarDecorator(new SimpleCoffee())));
```

## Decorator et principe Open/Closed

- **Ouvert à l'extension** : on ajoute un nouveau comportement en créant une nouvelle classe (un nouveau Concrete Decorator), sans toucher aux classes existantes.
- **Fermé à la modification** : on ne modifie ni le Concrete Component ni les anciens Decorators pour ajouter une option (ex. noisette, sirop).

Exemple d'extension :

```java
// Nouveau décorateur, sans modifier Coffee, SimpleCoffee ni les autres décorateurs
class WhipDecorator extends CoffeeDecorator {
    public WhipDecorator(Coffee coffee) { super(coffee); }
    public String getDescription() { return coffee.getDescription() + ", whip"; }
    public double getCost() { return coffee.getCost() + 0.7; }
}
```

## En résumé

| Élément            | Rôle principal                                      |
|--------------------|-----------------------------------------------------|
| **Component**      | Interface commune (composant + décorateurs)         |
| **Concrete Component** | Objet de base, comportement minimal            |
| **Decorator**      | Référence vers un Component, délégation            |
| **Concrete Decorator** | Une responsabilité ajoutée (coût, log, validation, etc.) |

Le cœur du pattern : **composition + même interface + délégation**. Le client voit toujours un Composant ; les décorateurs enveloppent ce composant et ajoutent du comportement de façon modulaire.

---

**Prochaine étape** : [Exemples concrets](./03-exemples-concrets.md) (café, logger, validation, flux).
