# Mise en pratique du pattern Decorator

Ce guide aide à décider quand utiliser le Decorator, comment l'appliquer pas à pas, et quelles alternatives ou pièges éviter.

## Quand utiliser le Decorator ?

### Utilisez le Decorator quand :

1. **Extensions dynamiques** : Vous voulez ajouter ou retirer des responsabilités à un objet **à l'exécution**, sans modifier sa classe.
2. **Combinaisons nombreuses** : Les combinaisons de fonctionnalités sont trop nombreuses pour être gérées par l'héritage (ex. café + lait + sucre + chantilly + sirop + …).
3. **Principe Open/Closed** : Vous voulez étendre le comportement en ajoutant de nouvelles classes (décorateurs) sans modifier les classes existantes.
4. **Même interface** : Le client doit pouvoir utiliser l'objet décoré exactement comme l'objet de base (même type, même contrat).
5. **Comportements empilables** : Les responsabilités peuvent s'empiler (logging + cache + compression, etc.) et l'ordre peut avoir du sens.

### Exemples de bons cas d'usage

- **Logging** : logger de base + timestamp + niveau + écriture fichier.
- **Validation** : validateur de base + requis + longueur + format email.
- **I/O** : flux de base + compression + chiffrement + buffer.
- **UI** : composant de base + bordure + défilement + barre de titre.
- **Prix / options** : produit de base + options (garantie, livraison express, etc.).

## Quand éviter le Decorator ?

### Évitez le Decorator quand :

1. **Comportement fixe et simple** : Une seule variante, pas d'empilement → une classe ou un paramètre suffit.
2. **Héritage suffisant** : Peu de combinaisons (2–3) et pas d'évolution prévue → sous-classes ou stratégies simples.
3. **Interface différente** : Le décoré doit exposer une API différente du composant → Adapter ou autre pattern.
4. **Ordre sans importance** : Si l'ordre des "couches" n'a pas de sens et que tout est optionnel de façon indépendante, une composition plus plate (objet avec options) peut suffire.
5. **Besoin d'enlever des décorateurs** : Le pattern ne prévoit pas facilement de retirer une couche une fois posée ; si vous devez activer/désactiver des comportements à la volée, une approche par stratégies ou par flags peut être plus simple.

### Alternatives à considérer

| Besoin | Alternative possible |
|--------|------------------------|
| Un seul comportement optionnel | Paramètre ou méthode optionnelle |
| Plusieurs algorithmes interchangeables | **Strategy** |
| Adapter une interface à une autre | **Adapter** |
| Contrôler l'accès (lazy, cache, proxy) | **Proxy** |
| Structure en arbre (partie / tout) | **Composite** |
| Beaucoup de paramètres optionnels | **Builder** ou objet de configuration |

## Étapes pour appliquer le pattern

### Étape 1 : Identifier le composant de base

- Quelle est l'entité que vous voulez étendre (logger, validateur, flux, produit) ?
- Quelle interface doit rester identique pour le client (méthodes publiques) ?

Définir une **interface (Composant)** et une **implémentation de base (Composant concret)**.

```java
interface Logger {
    void log(String message);
}

class BasicLogger implements Logger {
    public void log(String message) { System.out.println(message); }
}
```

### Étape 2 : Créer le Decorator de base

- Une classe qui implémente la même interface que le Composant.
- Un constructeur qui reçoit un Composant (celui qu'on décore).
- Les méthodes délèguent au composant décoré (sans ajouter de comportement pour l'instant).

```java
abstract class LoggerDecorator implements Logger {
    protected Logger logger;
    public LoggerDecorator(Logger logger) { this.logger = logger; }
    public void log(String message) { logger.log(message); }
}
```

### Étape 3 : Implémenter les Decorators concrets

- Une classe par responsabilité ajoutée (timestamp, niveau, fichier, etc.).
- Chaque décorateur hérite du Decorator de base, surcharge les méthodes concernées et appelle le composant décoré avant/après son travail.

```java
class TimestampDecorator extends LoggerDecorator {
    public TimestampDecorator(Logger logger) { super(logger); }
    public void log(String message) {
        String ts = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
        logger.log("[" + ts + "] " + message);
    }
}
```

### Étape 4 : Utiliser par composition

- Le client reçoit un type Composant (Logger, Coffee, etc.).
- Vous construisez l'objet en enchaînant les décorateurs autour du composant concret.

```java
Logger logger = new FileDecorator(
    new LevelDecorator(
        new TimestampDecorator(new BasicLogger()),
        "INFO"
    ),
    "app.log"
);
logger.log("Hello");
```

## Pièges à éviter

### 1. Décorateurs qui dépendent de l'ordre

L'ordre d'enveloppement peut changer le résultat (ex. log avant/après cache). Documenter l'ordre attendu ou le figer dans une factory/builder pour éviter les erreurs.

### 2. Trop de couches

Une chaîne très longue (A(B(C(D(E(...))))) peut nuire à la lisibilité et au débogage. Envisager des regroupements (ex. un décorateur "logging complet" qui encapsule timestamp + niveau + fichier) ou une structure plus haute niveau.

### 3. Décorateur qui casse la substitution

Le décorateur doit respecter le contrat du Composant (même signature, même sémantique). Éviter de lever des exceptions ou de changer le type de retour de façon inattendue.

### 4. Confusion Decorator / Proxy

- **Decorator** : ajoute ou modifie un comportement ; le but est d'enrichir.
- **Proxy** : contrôle l'accès (lazy, cache, vérification) ; le but est de garder la même interface tout en interceptant.

Un Proxy peut être implémenté comme un décorateur, mais l'intention (enrichissement vs contrôle d'accès) reste différente.

### 5. Utiliser le Decorator "pour faire bien"

N'introduire le pattern que si vous avez vraiment des extensions dynamiques, des combinaisons multiples ou un besoin clair d'Open/Closed. Sinon, une solution plus simple (paramètres, une ou deux sous-classes) est souvent préférable.

## Checklist rapide

Avant d'introduire un Decorator, vérifier :

- [ ] Les responsabilités à ajouter sont-elles **empilables** et **optionnelles** ?
- [ ] Y a-t-il **plusieurs combinaisons** (ou une évolution probable) ?
- [ ] Le client doit-il utiliser l'objet **sans savoir** s'il est décoré ou non ?
- [ ] Voulez-vous **étendre sans modifier** les classes existantes (OCP) ?

Si la majorité des réponses est oui, le Decorator est un bon candidat. Sinon, envisager une solution plus simple ou un autre pattern.

## Aller plus loin

- **[Design Patterns (patterns structurels)](../design-pattern/03-patterns-structurels.md)** : Comparaison avec Adapter, Proxy, Composite.
- **[Open/Closed (SOLID)](../solid/03-open-closed.md)** : Le Decorator comme application de l'OCP.
- **[Exemples concrets](./03-exemples-concrets.md)** : Revoir les exemples café, logger, validation, flux.

---

**Rappel** : Le Decorator est un outil pour étendre le comportement de façon modulaire. Utilisez-le lorsque les combinaisons de fonctionnalités sont nombreuses ou évolutives ; sinon, privilégiez la solution la plus simple.
