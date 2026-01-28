# Introduction aux Design Patterns

## Qu'est-ce qu'un Design Pattern ?

Un **design pattern** est une solution réutilisable à un problème récurrent dans la conception de logiciels. C'est une description ou un modèle pour résoudre un problème qui peut être utilisé dans de nombreuses situations différentes.

### Analogie avec l'architecture

Imaginez que vous construisez une maison. Au lieu de réinventer la roue à chaque fois, vous utilisez des plans éprouvés :
- **Maison avec garage** : solution pour le problème "où garer la voiture ?"
- **Maison avec grenier** : solution pour le problème "où stocker les affaires ?"

Les design patterns sont les "plans architecturaux" du développement logiciel.

## Pourquoi utiliser les Design Patterns ?

### Avantages

1. **Solutions éprouvées** : Ces patterns ont été testés et validés par des milliers de développeurs
2. **Communication** : Un nom commun facilite la communication entre développeurs
3. **Réduction de la complexité** : Évite de réinventer la roue
4. **Maintenabilité** : Code plus facile à comprendre et maintenir
5. **Flexibilité** : Facilite les modifications futures

### Inconvénients à éviter

1. **Surcharge** : Ne pas utiliser un pattern si le problème est simple
2. **Complexité inutile** : Un pattern mal appliqué complique le code
3. **Apprentissage** : Nécessite de comprendre le pattern avant de l'utiliser

## Structure d'un Design Pattern

Chaque pattern est généralement décrit avec :

1. **Nom** : Identifiant unique et mémorable
2. **Intention** : Ce que le pattern résout
3. **Problème** : Quand l'utiliser
4. **Solution** : Comment le pattern résout le problème
5. **Conséquences** : Avantages et inconvénients

## Classification des Patterns

### Patterns de Création (Creational)

**Problème résolu** : Comment créer des objets de manière flexible et réutilisable ?

**Exemple concret** : 
- Vous avez besoin d'une seule instance d'une classe (configuration, logger) → **Singleton**
- Vous ne savez pas à l'avance quel type d'objet créer → **Factory**

### Patterns Structurels (Structural)

**Problème résolu** : Comment composer des objets pour former des structures plus grandes ?

**Exemple concret** :
- Vous avez deux interfaces incompatibles à faire fonctionner ensemble → **Adapter**
- Vous voulez ajouter des fonctionnalités à un objet sans modifier sa classe → **Decorator**

### Patterns Comportementaux (Behavioral)

**Problème résolu** : Comment gérer la communication et les responsabilités entre objets ?

**Exemple concret** :
- Vous voulez être notifié quand un événement se produit → **Observer**
- Vous voulez encapsuler une requête comme un objet → **Command**

## Principes fondamentaux

### 1. Programmer vers une interface, pas une implémentation

```python
# ❌ Mauvais : dépendance à une implémentation concrète
class PaymentProcessor:
    def process(self, payment):
        stripe = Stripe()  # Couplage fort
        stripe.charge(payment)

# ✅ Bon : dépendance à une abstraction
class PaymentProcessor:
    def __init__(self, payment_gateway):
        self.payment_gateway = payment_gateway  # Interface
    
    def process(self, payment):
        self.payment_gateway.charge(payment)
```

### 2. Favoriser la composition sur l'héritage

```python
# ❌ Mauvais : héritage rigide
class Duck:
    def quack(self): pass
    def fly(self): pass

class RubberDuck(Duck):
    def fly(self):
        raise Exception("Rubber ducks can't fly!")  # Violation LSP

# ✅ Bon : composition flexible
class Duck:
    def __init__(self, quack_behavior, fly_behavior):
        self.quack_behavior = quack_behavior
        self.fly_behavior = fly_behavior
    
    def quack(self):
        self.quack_behavior.quack()
    
    def fly(self):
        self.fly_behavior.fly()
```

### 3. Encapsuler ce qui varie

```python
# ❌ Mauvais : logique conditionnelle répétée
class Order:
    def calculate_shipping(self, country):
        if country == "FR":
            return 5.0
        elif country == "US":
            return 10.0
        elif country == "JP":
            return 15.0
        # ...

# ✅ Bon : stratégie encapsulée
class ShippingStrategy:
    def calculate(self): pass

class FranceShipping(ShippingStrategy):
    def calculate(self):
        return 5.0

class Order:
    def __init__(self, shipping_strategy):
        self.shipping_strategy = shipping_strategy
    
    def calculate_shipping(self):
        return self.shipping_strategy.calculate()
```

## Quand utiliser un Design Pattern ?

### ✅ Utilisez un pattern quand :

1. **Vous avez un problème récurrent** : Le pattern résout exactement votre problème
2. **La complexité est justifiée** : Le pattern simplifie réellement votre code
3. **L'équipe connaît le pattern** : Tout le monde comprend la solution
4. **Vous avez besoin de flexibilité** : Le pattern facilite les changements futurs

### ❌ N'utilisez PAS un pattern quand :

1. **Le problème est simple** : Une solution directe suffit
2. **Vous ne comprenez pas le pattern** : Apprenez d'abord, utilisez ensuite
3. **C'est juste pour "faire bien"** : Les patterns ne sont pas des trophées
4. **Cela complique inutilement** : Si c'est plus simple sans pattern, ne l'utilisez pas

## Exemple : Évolution naturelle vers un pattern

### Étape 1 : Code simple (sans pattern)

```python
class Logger:
    def log(self, message):
        print(f"[LOG] {message}")

logger = Logger()
logger.log("Application started")
```

### Étape 2 : Besoin d'une seule instance

```python
class Logger:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
    
    def log(self, message):
        print(f"[LOG] {message}")

logger1 = Logger()
logger2 = Logger()
print(logger1 is logger2)  # True - même instance
```

**Vous venez d'appliquer le pattern Singleton !** Mais de manière naturelle, pour résoudre un problème réel.

## Anti-patterns : Ce qu'il faut éviter

### 1. God Object
Un objet qui fait trop de choses.

```python
# ❌ Mauvais
class Application:
    def handle_user_input(self): pass
    def process_payment(self): pass
    def send_email(self): pass
    def generate_report(self): pass
    def manage_database(self): pass
    # ... 50 autres méthodes
```

### 2. Spaghetti Code
Code sans structure, difficile à suivre.

### 3. Golden Hammer
Utiliser le même pattern partout, même quand ce n'est pas approprié.

## Prochaines étapes

Maintenant que vous comprenez les concepts de base, explorez les patterns spécifiques :

- **[Patterns de Création](./02-patterns-creationnels.md)** - Pour créer des objets intelligemment
- **[Patterns Structurels](./03-patterns-structurels.md)** - Pour organiser vos classes
- **[Patterns Comportementaux](./04-patterns-comportementaux.md)** - Pour gérer les interactions

---

**Rappel** : Les design patterns sont des outils, pas des objectifs. Utilisez-les pour résoudre des problèmes réels, pas pour impressionner vos collègues !
