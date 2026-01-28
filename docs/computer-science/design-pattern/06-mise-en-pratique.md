# Mise en Pratique

Ce guide vous aidera à identifier quand utiliser un design pattern et comment l'appliquer efficacement dans vos projets.

## Comment choisir un Design Pattern ?

### Processus de décision

1. **Identifier le problème** : Quel problème essayez-vous de résoudre ?
2. **Évaluer la complexité** : Le pattern simplifie-t-il vraiment votre code ?
3. **Considérer les alternatives** : Y a-t-il une solution plus simple ?
4. **Vérifier la cohérence** : Le pattern s'intègre-t-il avec votre architecture existante ?

### Questions à se poser

Avant d'appliquer un pattern, demandez-vous :

- ✅ Le problème est-il récurrent dans votre codebase ?
- ✅ Le pattern réduit-il la complexité ou l'augmente-t-il ?
- ✅ Votre équipe comprendra-t-elle le pattern ?
- ✅ Le pattern facilite-t-il les tests ?
- ✅ Le pattern améliore-t-il la maintenabilité ?

## Guide de sélection rapide

### Patterns de Création

| Problème | Pattern | Exemple |
|----------|---------|---------|
| Besoin d'une seule instance | **Singleton** | Logger, Configuration |
| Création dépend du contexte | **Factory Method** | Création de transport selon la destination |
| Famille d'objets liés | **Abstract Factory** | UI multi-plateforme |
| Objet avec nombreux paramètres | **Builder** | Requête HTTP, Configuration |
| Clonage d'objets coûteux | **Prototype** | Configurations multiples |

### Patterns Structurels

| Problème | Pattern | Exemple |
|----------|---------|---------|
| Interface incompatible | **Adapter** | Intégration API tierce |
| Ajouter fonctionnalités dynamiquement | **Decorator** | Validation, Logging |
| Simplifier un système complexe | **Facade** | API simplifiée |
| Contrôler l'accès à un objet | **Proxy** | Cache, Lazy loading |
| Structure hiérarchique | **Composite** | Menu, Système de fichiers |

### Patterns Comportementaux

| Problème | Pattern | Exemple |
|----------|---------|---------|
| Notifications d'événements | **Observer** | Système d'événements |
| Algorithmes interchangeables | **Strategy** | Calcul de remises, Tri |
| Encapsuler des requêtes | **Command** | Undo/Redo, Files d'attente |
| Traitement en chaîne | **Chain of Responsibility** | Validation, Gestion d'erreurs |
| Comportement selon l'état | **State** | Machine à états, Workflow |

## Étapes pour appliquer un pattern

### Étape 1 : Comprendre le problème

```python
# ❌ Code problématique
class PaymentProcessor:
    def process(self, amount, method):
        if method == "credit_card":
            # Logique carte de crédit
            pass
        elif method == "paypal":
            # Logique PayPal
            pass
        elif method == "bank_transfer":
            # Logique virement
            pass
        # ... de plus en plus de conditions
```

**Problème identifié** : Trop de conditions, difficile à étendre, violation du principe Open/Closed.

### Étape 2 : Choisir le pattern approprié

Pour ce cas : **Strategy Pattern** - différents algorithmes de paiement interchangeables.

### Étape 3 : Refactoriser progressivement

```python
# ✅ Solution avec Strategy
class PaymentStrategy(ABC):
    @abstractmethod
    def pay(self, amount: float):
        pass

class CreditCardStrategy(PaymentStrategy):
    def pay(self, amount: float):
        # Logique carte de crédit
        pass

class PaymentProcessor:
    def __init__(self, strategy: PaymentStrategy):
        self.strategy = strategy
    
    def process(self, amount: float):
        return self.strategy.pay(amount)
```

### Étape 4 : Tester et valider

```python
def test_payment_processor():
    processor = PaymentProcessor(CreditCardStrategy())
    assert processor.process(100.0) == True
```

## Anti-patterns à éviter

### 1. Pattern Overuse (Surcharge de patterns)

❌ **Mauvais** : Utiliser un pattern pour un problème simple

```python
# Trop complexe pour un cas simple
class SimpleCalculator:
    def __init__(self, strategy: AddStrategy):
        self.strategy = strategy
    
    def add(self, a, b):
        return self.strategy.execute(a, b)
```

✅ **Bon** : Solution directe

```python
class SimpleCalculator:
    def add(self, a, b):
        return a + b
```

### 2. Golden Hammer (Marteau d'or)

❌ **Mauvais** : Utiliser le même pattern partout

```python
# Utiliser Singleton pour tout
class UserService(Singleton): pass
class ProductService(Singleton): pass
class OrderService(Singleton): pass
```

✅ **Bon** : Choisir le pattern approprié pour chaque cas

### 3. Pattern Without Purpose (Pattern sans but)

❌ **Mauvais** : Appliquer un pattern "parce que c'est bien"

```python
# Factory pour créer un seul type d'objet simple
class UserFactory:
    @staticmethod
    def create(name: str):
        return User(name)
```

✅ **Bon** : Utiliser le pattern seulement si nécessaire

```python
# Simple et direct
user = User(name)
```

## Refactoring vers les patterns

### Scénario 1 : Code avec beaucoup de conditions

**Avant** :
```python
def calculate_shipping(country, weight):
    if country == "FR":
        if weight < 1:
            return 5.0
        elif weight < 5:
            return 10.0
        else:
            return 20.0
    elif country == "US":
        if weight < 1:
            return 8.0
        # ... beaucoup de conditions
```

**Après** (Strategy Pattern) :
```python
class ShippingStrategy(ABC):
    @abstractmethod
    def calculate(self, weight: float) -> float:
        pass

class FranceShipping(ShippingStrategy):
    def calculate(self, weight: float) -> float:
        if weight < 1:
            return 5.0
        elif weight < 5:
            return 10.0
        return 20.0

def calculate_shipping(strategy: ShippingStrategy, weight: float):
    return strategy.calculate(weight)
```

### Scénario 2 : Code dupliqué

**Avant** :
```python
class EmailLogger:
    def log(self, message):
        # Envoyer par email
        pass

class FileLogger:
    def log(self, message):
        # Écrire dans fichier
        pass

class DatabaseLogger:
    def log(self, message):
        # Écrire en base
        pass

# Utilisation avec duplication
email_logger = EmailLogger()
file_logger = FileLogger()
db_logger = DatabaseLogger()

email_logger.log("Error")
file_logger.log("Error")
db_logger.log("Error")
```

**Après** (Decorator Pattern) :
```python
class Logger(ABC):
    @abstractmethod
    def log(self, message: str):
        pass

class BaseLogger(Logger):
    def log(self, message: str):
        print(message)

class EmailLogger(Logger):
    def __init__(self, logger: Logger):
        self.logger = logger
    
    def log(self, message: str):
        self.logger.log(message)
        # Envoyer par email

# Utilisation chaînée
logger = EmailLogger(FileLogger(DatabaseLogger(BaseLogger())))
logger.log("Error")  # Log dans tous les systèmes
```

## Checklist avant d'appliquer un pattern

- [ ] Le problème est clairement identifié
- [ ] Le pattern résout réellement le problème
- [ ] La solution sans pattern serait plus complexe
- [ ] L'équipe comprend le pattern
- [ ] Le pattern facilite les tests
- [ ] Le pattern améliore la maintenabilité
- [ ] Le pattern s'intègre avec l'architecture existante
- [ ] La documentation est à jour

## Pratiques recommandées

### 1. Commencez simple

```python
# Commencez avec une solution simple
def process_order(order):
    # Logique simple
    pass

# Refactorisez vers un pattern seulement si nécessaire
class OrderProcessor:
    def process(self, order):
        # Avec pattern si la complexité augmente
        pass
```

### 2. Documentez vos choix

```python
class PaymentProcessor:
    """
    Utilise le Strategy Pattern pour permettre
    différents algorithmes de paiement.
    
    Raison : Nous devons supporter plusieurs
    méthodes de paiement et en ajouter facilement.
    """
    pass
```

### 3. Testez vos patterns

```python
def test_strategy_pattern():
    """Test que le Strategy Pattern fonctionne correctement"""
    strategy = CreditCardStrategy()
    processor = PaymentProcessor(strategy)
    assert processor.process(100.0) == True
```

### 4. Refactorisez progressivement

Ne refactorisez pas tout d'un coup. Procédez par étapes :

1. Identifiez le code problématique
2. Appliquez le pattern à une petite partie
3. Testez
4. Étendez progressivement
5. Documentez

## Exemples de refactoring progressif

### Étape 1 : Code initial

```python
class UserService:
    def create_user(self, name, email):
        # Validation
        if not name:
            raise ValueError("Name required")
        if "@" not in email:
            raise ValueError("Invalid email")
        
        # Création
        user = User(name, email)
        
        # Sauvegarde
        db.save(user)
        
        # Notification
        email_service.send_welcome(email)
        
        return user
```

### Étape 2 : Extraire la validation (Chain of Responsibility)

```python
class Validator(ABC):
    def __init__(self):
        self._next = None
    
    def set_next(self, validator):
        self._next = validator
    
    @abstractmethod
    def validate(self, data):
        if self._next:
            return self._next.validate(data)
        return True

class NameValidator(Validator):
    def validate(self, data):
        if not data.get("name"):
            raise ValueError("Name required")
        return super().validate(data)

class EmailValidator(Validator):
    def validate(self, data):
        if "@" not in data.get("email", ""):
            raise ValueError("Invalid email")
        return super().validate(data)

class UserService:
    def __init__(self):
        self.validator = NameValidator()
        self.validator.set_next(EmailValidator())
    
    def create_user(self, name, email):
        self.validator.validate({"name": name, "email": email})
        # ... reste du code
```

### Étape 3 : Extraire les notifications (Observer)

```python
class UserObserver(ABC):
    @abstractmethod
    def notify(self, event, data):
        pass

class EmailNotifier(UserObserver):
    def notify(self, event, data):
        if event == "user_created":
            email_service.send_welcome(data["email"])

class UserService:
    def __init__(self):
        self.observers = []
    
    def attach(self, observer):
        self.observers.append(observer)
    
    def create_user(self, name, email):
        # ... validation et création
        for observer in self.observers:
            observer.notify("user_created", {"email": email})
```

## Outils et ressources

### Outils de détection

- **Code smells** : Identifiez les problèmes avant d'appliquer des patterns
- **Linters** : Détectez la complexité cyclomatique
- **Tests de couverture** : Assurez-vous que vos refactorings ne cassent rien

### Ressources d'apprentissage

1. **Lisez le code existant** : Voyez comment les patterns sont utilisés dans des projets open-source
2. **Pratiquez** : Refactorisez votre propre code
3. **Code reviews** : Demandez des retours sur vos implémentations
4. **Pair programming** : Apprenez des autres développeurs

## Conclusion

Les design patterns sont des outils puissants, mais ils doivent être utilisés judicieusement :

✅ **Utilisez-les quand** :
- Ils résolvent un problème réel
- Ils simplifient votre code
- Ils améliorent la maintenabilité

❌ **Évitez-les quand** :
- Le problème est simple
- Ils ajoutent de la complexité inutile
- Vous ne comprenez pas le pattern

**Rappel** : Le meilleur code est celui qui est simple, lisible et maintenable. Les patterns sont un moyen d'y arriver, pas une fin en soi.

---

**Retour à l'index** : [README](./README.md)
