# Patterns de Création (Creational Patterns)

Les patterns de création résolvent les problèmes liés à la création d'objets. Ils fournissent des mécanismes flexibles pour instancier des classes, améliorant ainsi la réutilisabilité et la maintenabilité du code.

## 1. Singleton

### Intention

Garantir qu'une classe n'a qu'une seule instance et fournir un point d'accès global à cette instance.

### Problème résolu

Quand avez-vous besoin d'une seule instance ?
- Configuration globale de l'application
- Connexion à une base de données
- Logger
- Cache partagé

### Solution

```python
class DatabaseConnection:
    _instance = None
    _initialized = False
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
    
    def __init__(self):
        if not DatabaseConnection._initialized:
            self.connection_string = "postgresql://localhost/mydb"
            self.connect()
            DatabaseConnection._initialized = True
    
    def connect(self):
        print(f"Connecting to {self.connection_string}")
    
    def query(self, sql):
        print(f"Executing: {sql}")

# Utilisation
db1 = DatabaseConnection()
db2 = DatabaseConnection()
print(db1 is db2)  # True - même instance
```

### Version thread-safe (Python)

```python
import threading

class ThreadSafeSingleton:
    _instance = None
    _lock = threading.Lock()
    
    def __new__(cls):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
        return cls._instance
```

### Cas d'usage concret : Logger

```python
class Logger:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance.logs = []
        return cls._instance
    
    def log(self, level, message):
        log_entry = f"[{level}] {message}"
        self.logs.append(log_entry)
        print(log_entry)
    
    def get_logs(self):
        return self.logs

# Utilisation dans toute l'application
logger = Logger()
logger.log("INFO", "Application started")

# Dans un autre module
logger2 = Logger()
logger2.log("ERROR", "Something went wrong")
print(len(logger.get_logs()))  # 2 - même instance
```

### Quand utiliser Singleton ?

**Utilisez-le quand** :
- Vous avez besoin d'exactement une instance d'une classe
- Cette instance doit être accessible globalement
- L'instance doit être initialisée à la première utilisation

**Évitez-le quand** :
- Vous pouvez utiliser une dépendance injectée
- Vous avez besoin de plusieurs instances avec des configurations différentes
- Vous testez votre code (difficile à mocker)

---

## 2. Factory Method

### Intention

Définir une interface pour créer un objet, mais laisser les sous-classes décider quelle classe instancier.

### Problème résolu

Comment créer des objets sans spécifier leur classe exacte ?

### Solution

```python
from abc import ABC, abstractmethod

# Interface du produit
class Transport(ABC):
    @abstractmethod
    def deliver(self, destination: str):
        pass

# Implémentations concrètes
class Truck(Transport):
    def deliver(self, destination: str):
        print(f"Delivering by truck to {destination}")

class Ship(Transport):
    def deliver(self, destination: str):
        print(f"Delivering by ship to {destination}")

class Airplane(Transport):
    def deliver(self, destination: str):
        print(f"Delivering by airplane to {destination}")

# Créateur abstrait
class Logistics(ABC):
    @abstractmethod
    def create_transport(self) -> Transport:
        pass
    
    def plan_delivery(self, destination: str):
        transport = self.create_transport()
        transport.deliver(destination)

# Créateurs concrets
class RoadLogistics(Logistics):
    def create_transport(self) -> Transport:
        return Truck()

class SeaLogistics(Logistics):
    def create_transport(self) -> Transport:
        return Ship()

class AirLogistics(Logistics):
    def create_transport(self) -> Transport:
        return Airplane()

# Utilisation
road = RoadLogistics()
road.plan_delivery("Paris")  # Delivering by truck to Paris

sea = SeaLogistics()
sea.plan_delivery("Tokyo")  # Delivering by ship to Tokyo
```

### Cas d'usage concret : Système de paiement

```python
from abc import ABC, abstractmethod

class PaymentMethod(ABC):
    @abstractmethod
    def pay(self, amount: float):
        pass

class CreditCard(PaymentMethod):
    def pay(self, amount: float):
        print(f"Paying {amount}€ with credit card")

class PayPal(PaymentMethod):
    def pay(self, amount: float):
        print(f"Paying {amount}€ with PayPal")

class BankTransfer(PaymentMethod):
    def pay(self, amount: float):
        print(f"Paying {amount}€ via bank transfer")

class PaymentProcessor(ABC):
    @abstractmethod
    def create_payment_method(self) -> PaymentMethod:
        pass
    
    def process_payment(self, amount: float):
        payment = self.create_payment_method()
        payment.pay(amount)

class CreditCardProcessor(PaymentProcessor):
    def create_payment_method(self) -> PaymentMethod:
        return CreditCard()

class PayPalProcessor(PaymentProcessor):
    def create_payment_method(self) -> PaymentMethod:
        return PayPal()

# Utilisation
processor = CreditCardProcessor()
processor.process_payment(100.0)
```

### Quand utiliser Factory Method ?

**Utilisez-le quand** :
- Vous ne connaissez pas à l'avance les types exacts d'objets à créer
- Vous voulez permettre aux sous-classes de spécifier les objets à créer
- Vous voulez découpler le code de création du code métier

---

## 3. Abstract Factory

### Intention

Fournir une interface pour créer des familles d'objets liés sans spécifier leurs classes concrètes.

### Problème résolu

Comment créer des ensembles d'objets cohérents qui fonctionnent ensemble ?

### Solution

```python
from abc import ABC, abstractmethod

# Interfaces des produits
class Button(ABC):
    @abstractmethod
    def render(self):
        pass

class Checkbox(ABC):
    @abstractmethod
    def render(self):
        pass

# Produits Windows
class WindowsButton(Button):
    def render(self):
        print("Rendering Windows button")

class WindowsCheckbox(Checkbox):
    def render(self):
        print("Rendering Windows checkbox")

# Produits macOS
class MacOSButton(Button):
    def render(self):
        print("Rendering macOS button")

class MacOSCheckbox(Checkbox):
    def render(self):
        print("Rendering macOS checkbox")

# Factory abstraite
class GUIFactory(ABC):
    @abstractmethod
    def create_button(self) -> Button:
        pass
    
    @abstractmethod
    def create_checkbox(self) -> Checkbox:
        pass

# Factories concrètes
class WindowsFactory(GUIFactory):
    def create_button(self) -> Button:
        return WindowsButton()
    
    def create_checkbox(self) -> Checkbox:
        return WindowsCheckbox()

class MacOSFactory(GUIFactory):
    def create_button(self) -> Button:
        return MacOSButton()
    
    def create_checkbox(self) -> Checkbox:
        return MacOSCheckbox()

# Application
class Application:
    def __init__(self, factory: GUIFactory):
        self.factory = factory
        self.button = None
        self.checkbox = None
    
    def create_ui(self):
        self.button = self.factory.create_button()
        self.checkbox = self.factory.create_checkbox()
    
    def render(self):
        self.button.render()
        self.checkbox.render()

# Utilisation
import platform

if platform.system() == "Windows":
    factory = WindowsFactory()
else:
    factory = MacOSFactory()

app = Application(factory)
app.create_ui()
app.render()
```

### Cas d'usage concret : Système de base de données

```python
from abc import ABC, abstractmethod

class Connection(ABC):
    @abstractmethod
    def connect(self):
        pass

class Query(ABC):
    @abstractmethod
    def execute(self, sql: str):
        pass

# PostgreSQL
class PostgreSQLConnection(Connection):
    def connect(self):
        print("Connecting to PostgreSQL")

class PostgreSQLQuery(Query):
    def execute(self, sql: str):
        print(f"PostgreSQL: {sql}")

# MySQL
class MySQLConnection(Connection):
    def connect(self):
        print("Connecting to MySQL")

class MySQLQuery(Query):
    def execute(self, sql: str):
        print(f"MySQL: {sql}")

# Factory abstraite
class DatabaseFactory(ABC):
    @abstractmethod
    def create_connection(self) -> Connection:
        pass
    
    @abstractmethod
    def create_query(self) -> Query:
        pass

class PostgreSQLFactory(DatabaseFactory):
    def create_connection(self) -> Connection:
        return PostgreSQLConnection()
    
    def create_query(self) -> Query:
        return PostgreSQLQuery()

class MySQLFactory(DatabaseFactory):
    def create_connection(self) -> Connection:
        return MySQLConnection()
    
    def create_query(self) -> Query:
        return MySQLQuery()

# Utilisation
def setup_database(db_type: str):
    if db_type == "postgresql":
        factory = PostgreSQLFactory()
    elif db_type == "mysql":
        factory = MySQLFactory()
    else:
        raise ValueError(f"Unknown database: {db_type}")
    
    connection = factory.create_connection()
    query = factory.create_query()
    
    connection.connect()
    query.execute("SELECT * FROM users")
```

### Différence avec Factory Method

- **Factory Method** : Crée UN type d'objet
- **Abstract Factory** : Crée une FAMILLE d'objets liés

---

## 4. Builder

### Intention

Construire des objets complexes étape par étape. Permet de créer différentes représentations d'un objet avec le même code de construction.

### Problème résolu

Comment créer des objets complexes avec de nombreux paramètres optionnels sans avoir des constructeurs avec trop de paramètres ?

### Solution

```python
class Pizza:
    def __init__(self):
        self.size = None
        self.cheese = False
        self.pepperoni = False
        self.bacon = False
        self.mushrooms = False
    
    def __str__(self):
        toppings = []
        if self.cheese:
            toppings.append("cheese")
        if self.pepperoni:
            toppings.append("pepperoni")
        if self.bacon:
            toppings.append("bacon")
        if self.mushrooms:
            toppings.append("mushrooms")
        
        return f"Pizza(size={self.size}, toppings={', '.join(toppings)})"

class PizzaBuilder:
    def __init__(self):
        self.pizza = Pizza()
    
    def set_size(self, size: str):
        self.pizza.size = size
        return self
    
    def add_cheese(self):
        self.pizza.cheese = True
        return self
    
    def add_pepperoni(self):
        self.pizza.pepperoni = True
        return self
    
    def add_bacon(self):
        self.pizza.bacon = True
        return self
    
    def add_mushrooms(self):
        self.pizza.mushrooms = True
        return self
    
    def build(self):
        if not self.pizza.size:
            raise ValueError("Pizza size is required")
        return self.pizza

# Utilisation
builder = PizzaBuilder()
pizza = (builder
         .set_size("large")
         .add_cheese()
         .add_pepperoni()
         .add_mushrooms()
         .build())

print(pizza)  # Pizza(size=large, toppings=cheese, pepperoni, mushrooms)
```

### Cas d'usage concret : Requête HTTP

```python
class HttpRequest:
    def __init__(self):
        self.method = None
        self.url = None
        self.headers = {}
        self.body = None
        self.timeout = 30
    
    def __str__(self):
        return f"{self.method} {self.url} (timeout: {self.timeout}s)"

class HttpRequestBuilder:
    def __init__(self):
        self.request = HttpRequest()
    
    def set_method(self, method: str):
        self.request.method = method.upper()
        return self
    
    def set_url(self, url: str):
        self.request.url = url
        return self
    
    def add_header(self, key: str, value: str):
        self.request.headers[key] = value
        return self
    
    def set_body(self, body: str):
        self.request.body = body
        return self
    
    def set_timeout(self, seconds: int):
        self.request.timeout = seconds
        return self
    
    def build(self):
        if not self.request.method:
            raise ValueError("HTTP method is required")
        if not self.request.url:
            raise ValueError("URL is required")
        return self.request

# Utilisation
request = (HttpRequestBuilder()
           .set_method("POST")
           .set_url("https://api.example.com/users")
           .add_header("Content-Type", "application/json")
           .add_header("Authorization", "Bearer token123")
           .set_body('{"name": "John"}')
           .set_timeout(60)
           .build())

print(request)
```

### Builder avec Director (optionnel)

```python
class PizzaDirector:
    def __init__(self, builder: PizzaBuilder):
        self.builder = builder
    
    def make_margherita(self):
        return (self.builder
                .set_size("medium")
                .add_cheese()
                .build())
    
    def make_pepperoni(self):
        return (self.builder
                .set_size("large")
                .add_cheese()
                .add_pepperoni()
                .build())
    
    def make_supreme(self):
        return (self.builder
                .set_size("large")
                .add_cheese()
                .add_pepperoni()
                .add_bacon()
                .add_mushrooms()
                .build())

# Utilisation
builder = PizzaBuilder()
director = PizzaDirector(builder)

margherita = director.make_margherita()
print(margherita)
```

### Quand utiliser Builder ?

**Utilisez-le quand** :
- Vous avez des objets avec de nombreux paramètres optionnels
- Vous voulez créer différentes représentations du même objet
- Vous voulez que la construction soit lisible et fluide

**Évitez-le quand** :
- L'objet est simple avec peu de paramètres
- Vous avez besoin d'immutabilité (utilisez plutôt des objets valeur)

---

## 5. Prototype

### Intention

Créer de nouveaux objets en copiant des instances existantes (prototypes), plutôt que de créer des objets à partir de zéro.

### Problème résolu

Comment créer des objets coûteux en termes de performance ou avec une configuration complexe ?

### Solution

```python
import copy
from abc import ABC, abstractmethod

class Shape(ABC):
    def __init__(self):
        self.x = 0
        self.y = 0
        self.color = None
    
    @abstractmethod
    def clone(self):
        pass
    
    def __str__(self):
        return f"{self.__class__.__name__}(x={self.x}, y={self.y}, color={self.color})"

class Circle(Shape):
    def __init__(self, radius=0):
        super().__init__()
        self.radius = radius
    
    def clone(self):
        return copy.deepcopy(self)

class Rectangle(Shape):
    def __init__(self, width=0, height=0):
        super().__init__()
        self.width = width
        self.height = height
    
    def clone(self):
        return copy.deepcopy(self)

# Registry de prototypes
class ShapeRegistry:
    def __init__(self):
        self.prototypes = {}
    
    def register(self, name: str, prototype: Shape):
        self.prototypes[name] = prototype
    
    def create(self, name: str) -> Shape:
        if name not in self.prototypes:
            raise ValueError(f"Unknown shape: {name}")
        return self.prototypes[name].clone()

# Utilisation
registry = ShapeRegistry()

# Créer et enregistrer des prototypes
circle_prototype = Circle(radius=10)
circle_prototype.color = "red"
registry.register("red_circle", circle_prototype)

rectangle_prototype = Rectangle(width=20, height=30)
rectangle_prototype.color = "blue"
registry.register("blue_rectangle", rectangle_prototype)

# Cloner les prototypes
circle1 = registry.create("red_circle")
circle2 = registry.create("red_circle")

circle1.x = 100
circle2.x = 200

print(circle1)  # Circle(x=100, y=0, color=red)
print(circle2)  # Circle(x=200, y=0, color=red)
print(circle1.radius == circle2.radius)  # True
```

### Cas d'usage concret : Configuration d'application

```python
import copy

class AppConfig:
    def __init__(self):
        self.database_url = None
        self.api_key = None
        self.timeout = 30
        self.debug = False
        self.features = {}
    
    def clone(self):
        return copy.deepcopy(self)
    
    def __str__(self):
        return f"Config(db={self.database_url}, timeout={self.timeout}, debug={self.debug})"

# Configuration par défaut
default_config = AppConfig()
default_config.database_url = "postgresql://localhost/app"
default_config.timeout = 30
default_config.debug = False

# Configuration de développement
dev_config = default_config.clone()
dev_config.database_url = "postgresql://localhost/app_dev"
dev_config.debug = True
dev_config.features["experimental"] = True

# Configuration de production
prod_config = default_config.clone()
prod_config.database_url = "postgresql://prod-server/app"
prod_config.debug = False
prod_config.timeout = 60

print(default_config)
print(dev_config)
print(prod_config)
```

### Quand utiliser Prototype ?

**Utilisez-le quand** :
- La création d'objets est coûteuse (requêtes DB, calculs complexes)
- Vous avez besoin de nombreuses variantes d'un objet
- Vous voulez éviter une hiérarchie de classes complexe

**Évitez-le quand** :
- Les objets sont simples à créer
- Vous avez peu d'instances à créer

---

## Résumé des Patterns de Création

| Pattern | Problème résolu | Quand l'utiliser |
|---------|----------------|------------------|
| **Singleton** | Une seule instance nécessaire | Configuration, logger, connexion DB |
| **Factory Method** | Création flexible d'un type d'objet | Dépend du contexte d'exécution |
| **Abstract Factory** | Création de familles d'objets | UI multi-plateforme, systèmes DB |
| **Builder** | Objets complexes avec nombreux paramètres | Requêtes HTTP, objets de configuration |
| **Prototype** | Clonage d'objets coûteux | Jeux vidéo, configurations multiples |

---

**Prochaine étape** : Découvrez les [Patterns Structurels](./03-patterns-structurels.md) pour apprendre à organiser vos classes efficacement.
