# Introduction au pattern Decorator

## Qu'est-ce que le pattern Decorator ?

Le **Decorator** est un pattern structurel qui permet d'**attacher dynamiquement** des responsabilités supplémentaires à un objet. Au lieu de modifier la classe existante ou d'empiler les sous-classes par héritage, on enveloppe l'objet dans des "décorateurs" qui ajoutent ou modifient son comportement.

### Intention (GoF)

> Attacher dynamiquement des responsabilités supplémentaires à un objet. Les decorators offrent une alternative souple à l'héritage pour étendre les fonctionnalités.

En résumé : **étendre un objet à l'exécution**, sans toucher à son code et sans explosion de sous-classes.

## Le problème : pourquoi pas l'héritage ?

### Exemple problématique

Imaginons un système de **café** : un café simple, avec lait, avec sucre, avec chantilly, etc.

**Avec l'héritage seul** :

```java
interface Coffee {
    double getCost();
}

class SimpleCoffee implements Coffee {
    public double getCost() { return 2.0; }
}

class CoffeeWithMilk implements Coffee {
    public double getCost() { return 2.5; }
}

class CoffeeWithSugar implements Coffee {
    public double getCost() { return 2.2; }
}

class CoffeeWithMilkAndSugar implements Coffee {  // Combinaison 1
    public double getCost() { return 2.7; }
}

class CoffeeWithMilkAndWhip implements Coffee {    // Combinaison 2
    public double getCost() { return 3.2; }
}

// Et ainsi de suite : Milk+Sugar+Whip, Sugar+Whip, ...
```

**Problèmes** :
- **Explosion de classes** : 3 options donnent déjà 7 combinaisons ; 5 options en donnent 31.
- **Duplication** : La logique "coût du lait" ou "coût du sucre" est répétée dans chaque combinaison.
- **Rigidité** : Chaque nouvelle option (noisette, sirop) oblige à créer de nouvelles sous-classes.
- **Violation Open/Closed** : On modifie ou on multiplie les classes au lieu d'étendre par composition.

### Ce qu'apporte le Decorator

Avec le Decorator, on part d'un **composant de base** (un café simple) et on l'enveloppe dans des décorateurs (lait, sucre, chantilly). Chaque décorateur ajoute son coût et sa description, et délègue au composant qu'il enveloppe.

```java
// Composant de base
Coffee coffee = new SimpleCoffee();

// On enveloppe : café → café + lait → café + lait + sucre
coffee = new MilkDecorator(coffee);
coffee = new SugarDecorator(coffee);

// Même interface : getCost(), getDescription()
System.out.println(coffee.getDescription());  // "Simple coffee, milk, sugar"
System.out.println(coffee.getCost());          // 2.7
```

**Avantages** :
- **Une classe par option** : `MilkDecorator`, `SugarDecorator`, `WhipDecorator`, etc.
- **Combinaisons illimitées** à l'exécution, sans nouvelle classe.
- **Ouvert à l'extension** (nouveau décorateur), **fermé à la modification** (on ne touche pas à `SimpleCoffee`).

## Analogie du quotidien

Pensez à un **vêtement** :
- La personne est le "composant de base".
- Chaque vêtement (t-shirt, pull, manteau) est un "décorateur" : il enveloppe ce qui est en dessous et ajoute une couche (chaleur, style, protection).
- On peut enchaîner les couches dans n'importe quel ordre.
- La personne ne change pas ; c’est la composition des couches qui définit le résultat.

Le Decorator fonctionne de la même façon : un objet de base enveloppé par plusieurs couches de comportement.

## Relation avec les principes SOLID

### Principe Open/Closed (OCP)

Le Decorator illustre bien l’OCP :
- **Ouvert à l’extension** : on ajoute de nouveaux décorateurs sans modifier les classes existantes.
- **Fermé à la modification** : le composant concret et les décorateurs déjà en place restent inchangés.

### Favoriser la composition sur l'héritage

Au lieu d’hériter pour ajouter du comportement (sous-classes multiples), on **compose** : un décorateur contient une référence vers le composant qu’il décore et délègue avant ou après avoir fait son travail.

## Quand le pattern Decorator est utile

- Vous voulez **ajouter ou retirer des responsabilités** à un objet **à l’exécution**.
- L’héritage serait lourd (trop de combinaisons) ou peu adapté (comportement optionnel, empilable).
- Vous voulez garder la **même interface** pour le client (le décoré reste utilisable comme le composant de base).
- Vous avez des besoins comme : logging, cache, compression, chiffrement, validation en chaîne, formatage de flux, etc.

## Ce que le Decorator n'est pas

- **Pas un simple wrapper** : le décorateur implémente la même interface (ou sous-type) que le composant et délègue les appels ; le client ne sait pas qu’il a un décorateur.
- **Pas une chaîne de responsabilité** : ici, chaque décorateur enveloppe un seul composant et ajoute un comportement ; la Chaîne de responsabilité fait passer une requête le long d’une chaîne de handlers.
- **Pas du remplacement d’objet** : on enrichit le même "flux" d’appels (même interface), pas un autre pattern comme Strategy ou Proxy (même si Proxy peut ressembler à un décorateur dans certains cas).

## Prochaines étapes

La suite de la documentation détaille :
- **[Structure et principe](./02-structure-et-principe.md)** : composant, décorateur de base, décorateurs concrets, et principe de délégation
- **[Exemples concrets](./03-exemples-concrets.md)** : café, logger, validation, flux
- **[Mise en pratique](./04-mise-en-pratique.md)** : quand l’utiliser, alternatives, pièges à éviter

---

**Rappel** : Le Decorator est une solution à un problème précis (extension dynamique par composition). Utilisez-le lorsque les combinaisons de comportements sont nombreuses ou évolutives, pas par simple envie d’appliquer un pattern.
