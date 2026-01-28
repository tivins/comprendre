# Patterns Structurels (Structural Patterns)

Les patterns structurels expliquent comment assembler des objets et des classes pour former des structures plus grandes, tout en gardant ces structures flexibles et efficaces.

## 1. Adapter

### Intention

Permettre à des interfaces incompatibles de travailler ensemble. L'Adapter agit comme un pont entre deux interfaces incompatibles.

### Problème résolu

Comment utiliser une classe existante dont l'interface ne correspond pas à ce dont vous avez besoin ?

### Solution

```python
# Interface cible (ce que vous voulez)
class MediaPlayer:
    def play(self, audio_type: str, filename: str):
        pass

# Interface existante (incompatible)
class AdvancedMediaPlayer:
    def play_vlc(self, filename: str):
        print(f"Playing VLC file: {filename}")
    
    def play_mp4(self, filename: str):
        print(f"Playing MP4 file: {filename}")

# Adapter
class MediaAdapter(MediaPlayer):
    def __init__(self, audio_type: str):
        self.advanced_player = AdvancedMediaPlayer()
        self.audio_type = audio_type
    
    def play(self, audio_type: str, filename: str):
        if audio_type == "vlc":
            self.advanced_player.play_vlc(filename)
        elif audio_type == "mp4":
            self.advanced_player.play_mp4(filename)

# Client utilisant l'interface cible
class AudioPlayer(MediaPlayer):
    def play(self, audio_type: str, filename: str):
        if audio_type == "mp3":
            print(f"Playing MP3 file: {filename}")
        elif audio_type in ["vlc", "mp4"]:
            adapter = MediaAdapter(audio_type)
            adapter.play(audio_type, filename)
        else:
            print(f"Invalid media type: {audio_type}")

# Utilisation
player = AudioPlayer()
player.play("mp3", "song.mp3")
player.play("mp4", "movie.mp4")
player.play("vlc", "video.vlc")
```

### Cas d'usage concret : API de paiement

```python
# API externe (incompatible avec votre code)
class StripeAPI:
    def charge_customer(self, customer_id: str, amount_cents: int):
        print(f"Stripe: Charging {amount_cents} cents to customer {customer_id}")

class PayPalAPI:
    def make_payment(self, email: str, amount_dollars: float):
        print(f"PayPal: Charging ${amount_dollars} to {email}")

# Interface que vous voulez utiliser
class PaymentProcessor:
    def process_payment(self, amount: float, currency: str, customer_info: dict):
        pass

# Adapters
class StripeAdapter(PaymentProcessor):
    def __init__(self):
        self.stripe = StripeAPI()
    
    def process_payment(self, amount: float, currency: str, customer_info: dict):
        # Convertir en cents (format Stripe)
        amount_cents = int(amount * 100)
        customer_id = customer_info.get("stripe_id")
        self.stripe.charge_customer(customer_id, amount_cents)

class PayPalAdapter(PaymentProcessor):
    def __init__(self):
        self.paypal = PayPalAPI()
    
    def process_payment(self, amount: float, currency: str, customer_info: dict):
        # Convertir en dollars (format PayPal)
        email = customer_info.get("email")
        self.paypal.make_payment(email, amount)

# Utilisation
def process_order(payment_processor: PaymentProcessor, amount: float):
    payment_processor.process_payment(
        amount=amount,
        currency="EUR",
        customer_info={"stripe_id": "cus_123", "email": "user@example.com"}
    )

# Utiliser avec Stripe
stripe_adapter = StripeAdapter()
process_order(stripe_adapter, 100.0)

# Utiliser avec PayPal
paypal_adapter = PayPalAdapter()
process_order(paypal_adapter, 100.0)
```

### Quand utiliser Adapter ?

✅ **Utilisez-le quand** :
- Vous voulez utiliser une classe existante dont l'interface ne correspond pas
- Vous intégrez des bibliothèques tierces
- Vous voulez réutiliser du code legacy

---

## 2. Decorator

### Intention

Attacher dynamiquement de nouvelles responsabilités à un objet. Le Decorator fournit une alternative flexible à l'héritage pour étendre les fonctionnalités.

### Problème résolu

Comment ajouter des fonctionnalités à un objet sans modifier sa classe ou utiliser l'héritage ?

### Solution

```python
from abc import ABC, abstractmethod

# Composant de base
class Coffee(ABC):
    @abstractmethod
    def get_description(self) -> str:
        pass
    
    @abstractmethod
    def get_cost(self) -> float:
        pass

# Composant concret
class SimpleCoffee(Coffee):
    def get_description(self) -> str:
        return "Simple coffee"
    
    def get_cost(self) -> float:
        return 2.0

# Decorator de base
class CoffeeDecorator(Coffee):
    def __init__(self, coffee: Coffee):
        self._coffee = coffee
    
    def get_description(self) -> str:
        return self._coffee.get_description()
    
    def get_cost(self) -> float:
        return self._coffee.get_cost()

# Decorators concrets
class MilkDecorator(CoffeeDecorator):
    def get_description(self) -> str:
        return self._coffee.get_description() + ", milk"
    
    def get_cost(self) -> float:
        return self._coffee.get_cost() + 0.5

class SugarDecorator(CoffeeDecorator):
    def get_description(self) -> str:
        return self._coffee.get_description() + ", sugar"
    
    def get_cost(self) -> float:
        return self._coffee.get_cost() + 0.2

class WhipDecorator(CoffeeDecorator):
    def get_description(self) -> str:
        return self._coffee.get_description() + ", whip"
    
    def get_cost(self) -> float:
        return self._coffee.get_cost() + 0.7

# Utilisation
coffee = SimpleCoffee()
print(f"{coffee.get_description()}: ${coffee.get_cost()}")

coffee_with_milk = MilkDecorator(coffee)
print(f"{coffee_with_milk.get_description()}: ${coffee_with_milk.get_cost()}")

coffee_with_milk_and_sugar = SugarDecorator(coffee_with_milk)
print(f"{coffee_with_milk_and_sugar.get_description()}: ${coffee_with_milk_and_sugar.get_cost()}")

# Peut aussi être chaîné directement
fancy_coffee = WhipDecorator(SugarDecorator(MilkDecorator(SimpleCoffee())))
print(f"{fancy_coffee.get_description()}: ${fancy_coffee.get_cost()}")
```

### Cas d'usage concret : Système de logging

```python
from abc import ABC, abstractmethod
from datetime import datetime

class Logger(ABC):
    @abstractmethod
    def log(self, message: str):
        pass

class BasicLogger(Logger):
    def log(self, message: str):
        print(message)

class LoggerDecorator(Logger):
    def __init__(self, logger: Logger):
        self._logger = logger
    
    def log(self, message: str):
        self._logger.log(message)

class TimestampDecorator(LoggerDecorator):
    def log(self, message: str):
        timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        self._logger.log(f"[{timestamp}] {message}")

class LevelDecorator(LoggerDecorator):
    def __init__(self, logger: Logger, level: str):
        super().__init__(logger)
        self.level = level
    
    def log(self, message: str):
        self._logger.log(f"[{self.level}] {message}")

class FileDecorator(LoggerDecorator):
    def __init__(self, logger: Logger, filename: str):
        super().__init__(logger)
        self.filename = filename
    
    def log(self, message: str):
        self._logger.log(message)
        with open(self.filename, "a") as f:
            f.write(message + "\n")

# Utilisation
basic_logger = BasicLogger()
basic_logger.log("Hello")  # Hello

# Ajouter timestamp
timestamp_logger = TimestampDecorator(basic_logger)
timestamp_logger.log("Hello")  # [2026-01-28 10:30:45] Hello

# Ajouter niveau
info_logger = LevelDecorator(timestamp_logger, "INFO")
info_logger.log("Hello")  # [INFO] [2026-01-28 10:30:45] Hello

# Ajouter écriture fichier
file_logger = FileDecorator(info_logger, "app.log")
file_logger.log("Hello")  # Affiche ET écrit dans app.log
```

### Cas d'usage concret : Validation de données

```python
from abc import ABC, abstractmethod

class Validator(ABC):
    @abstractmethod
    def validate(self, value: str) -> tuple[bool, str]:
        pass

class BaseValidator(Validator):
    def validate(self, value: str) -> tuple[bool, str]:
        return True, ""

class ValidatorDecorator(Validator):
    def __init__(self, validator: Validator):
        self._validator = validator
    
    def validate(self, value: str) -> tuple[bool, str]:
        return self._validator.validate(value)

class RequiredValidator(ValidatorDecorator):
    def validate(self, value: str) -> tuple[bool, str]:
        if not value:
            return False, "Field is required"
        return self._validator.validate(value)

class MinLengthValidator(ValidatorDecorator):
    def __init__(self, validator: Validator, min_length: int):
        super().__init__(validator)
        self.min_length = min_length
    
    def validate(self, value: str) -> tuple[bool, str]:
        if len(value) < self.min_length:
            return False, f"Minimum length is {self.min_length}"
        return self._validator.validate(value)

class EmailValidator(ValidatorDecorator):
    def validate(self, value: str) -> tuple[bool, str]:
        if "@" not in value:
            return False, "Invalid email format"
        return self._validator.validate(value)

# Utilisation
email_validator = EmailValidator(
    MinLengthValidator(
        RequiredValidator(BaseValidator()),
        min_length=5
    )
)

is_valid, error = email_validator.validate("")
print(f"Valid: {is_valid}, Error: {error}")  # Valid: False, Error: Field is required

is_valid, error = email_validator.validate("ab")
print(f"Valid: {is_valid}, Error: {error}")  # Valid: False, Error: Minimum length is 5

is_valid, error = email_validator.validate("user@example.com")
print(f"Valid: {is_valid}, Error: {error}")  # Valid: True, Error: 
```

### Quand utiliser Decorator ?

✅ **Utilisez-le quand** :
- Vous voulez ajouter des fonctionnalités dynamiquement
- L'héritage n'est pas approprié (trop de combinaisons possibles)
- Vous voulez respecter le principe Open/Closed

❌ **Évitez-le quand** :
- Les fonctionnalités sont fixes et connues à l'avance
- Vous pouvez utiliser l'héritage simple

---

## 3. Facade

### Intention

Fournir une interface unifiée et simplifiée à un ensemble d'interfaces dans un sous-système. Facade définit une interface de niveau supérieur qui facilite l'utilisation du sous-système.

### Problème résolu

Comment simplifier l'utilisation d'un système complexe avec de nombreuses classes interdépendantes ?

### Solution

```python
# Sous-système complexe
class CPU:
    def freeze(self):
        print("CPU: Freezing...")
    
    def jump(self, position: int):
        print(f"CPU: Jumping to {position}")
    
    def execute(self):
        print("CPU: Executing...")

class Memory:
    def load(self, position: int, data: str):
        print(f"Memory: Loading data '{data}' at position {position}")

class HardDrive:
    def read(self, lba: int, size: int) -> str:
        print(f"HardDrive: Reading {size} bytes from LBA {lba}")
        return "boot_data"

# Facade simplifiée
class ComputerFacade:
    def __init__(self):
        self.cpu = CPU()
        self.memory = Memory()
        self.hard_drive = HardDrive()
    
    def start(self):
        print("=== Starting computer ===")
        self.cpu.freeze()
        self.memory.load(0, self.hard_drive.read(0, 1024))
        self.cpu.jump(0)
        self.cpu.execute()
        print("=== Computer started ===\n")

# Utilisation
computer = ComputerFacade()
computer.start()

# Au lieu de :
# cpu = CPU()
# memory = Memory()
# hard_drive = HardDrive()
# cpu.freeze()
# memory.load(0, hard_drive.read(0, 1024))
# cpu.jump(0)
# cpu.execute()
```

### Cas d'usage concret : API de média sociale

```python
# Sous-système complexe
class UserService:
    def create_user(self, username: str, email: str):
        print(f"Creating user: {username} ({email})")
        return f"user_{username}"

class PostService:
    def create_post(self, user_id: str, content: str):
        print(f"Creating post for user {user_id}: {content}")
        return f"post_{user_id}"

class NotificationService:
    def send_welcome_email(self, email: str):
        print(f"Sending welcome email to {email}")
    
    def notify_followers(self, user_id: str):
        print(f"Notifying followers of {user_id}")

class AnalyticsService:
    def track_event(self, event_type: str, user_id: str):
        print(f"Tracking event: {event_type} for user {user_id}")

# Facade
class SocialMediaFacade:
    def __init__(self):
        self.user_service = UserService()
        self.post_service = PostService()
        self.notification_service = NotificationService()
        self.analytics_service = AnalyticsService()
    
    def register_user(self, username: str, email: str):
        """Inscription simplifiée d'un utilisateur"""
        print("=== Registering new user ===")
        user_id = self.user_service.create_user(username, email)
        self.notification_service.send_welcome_email(email)
        self.analytics_service.track_event("user_registered", user_id)
        print("=== User registered ===\n")
        return user_id
    
    def create_and_share_post(self, user_id: str, content: str):
        """Création et partage simplifié d'un post"""
        print("=== Creating and sharing post ===")
        post_id = self.post_service.create_post(user_id, content)
        self.notification_service.notify_followers(user_id)
        self.analytics_service.track_event("post_created", user_id)
        print("=== Post shared ===\n")
        return post_id

# Utilisation simplifiée
social_media = SocialMediaFacade()
user_id = social_media.register_user("john_doe", "john@example.com")
social_media.create_and_share_post(user_id, "Hello, world!")
```

### Quand utiliser Facade ?

✅ **Utilisez-le quand** :
- Vous voulez simplifier l'utilisation d'un sous-système complexe
- Vous voulez découpler le client du sous-système
- Vous voulez créer une couche d'abstraction

---

## 4. Proxy

### Intention

Fournir un substitut ou un espace réservé pour un autre objet pour contrôler l'accès à celui-ci.

### Problème résolu

Comment contrôler l'accès à un objet coûteux ou sensible sans modifier son code ?

### Types de Proxy

#### 1. Virtual Proxy (Lazy Loading)

```python
from abc import ABC, abstractmethod

class Image(ABC):
    @abstractmethod
    def display(self):
        pass

class RealImage(Image):
    def __init__(self, filename: str):
        self.filename = filename
        self.load_from_disk()
    
    def load_from_disk(self):
        print(f"Loading {self.filename} from disk...")
    
    def display(self):
        print(f"Displaying {self.filename}")

class ImageProxy(Image):
    def __init__(self, filename: str):
        self.filename = filename
        self.real_image = None
    
    def display(self):
        if self.real_image is None:
            self.real_image = RealImage(self.filename)
        self.real_image.display()

# Utilisation
print("=== Without Proxy ===")
image1 = RealImage("photo1.jpg")  # Charge immédiatement

print("\n=== With Proxy ===")
image2 = ImageProxy("photo2.jpg")  # Ne charge pas encore
image2.display()  # Charge maintenant
```

#### 2. Protection Proxy

```python
from abc import ABC, abstractmethod

class Database(ABC):
    @abstractmethod
    def query(self, sql: str):
        pass

class RealDatabase(Database):
    def query(self, sql: str):
        print(f"Executing: {sql}")
        return f"Results for: {sql}"

class DatabaseProxy(Database):
    def __init__(self, database: Database, user_role: str):
        self.database = database
        self.user_role = user_role
    
    def query(self, sql: str):
        # Vérifier les permissions
        if "DELETE" in sql.upper() and self.user_role != "admin":
            raise PermissionError("Only admins can delete")
        if "DROP" in sql.upper() and self.user_role != "admin":
            raise PermissionError("Only admins can drop tables")
        
        return self.database.query(sql)

# Utilisation
real_db = RealDatabase()
admin_db = DatabaseProxy(real_db, "admin")
user_db = DatabaseProxy(real_db, "user")

admin_db.query("SELECT * FROM users")
admin_db.query("DELETE FROM users WHERE id=1")  # OK pour admin

user_db.query("SELECT * FROM users")
try:
    user_db.query("DELETE FROM users WHERE id=1")  # Erreur pour user
except PermissionError as e:
    print(f"Error: {e}")
```

#### 3. Remote Proxy

```python
import json

# Objet distant (simulé)
class RemoteService:
    def __init__(self, url: str):
        self.url = url
    
    def fetch_data(self, endpoint: str):
        # Simulation d'un appel réseau
        print(f"Fetching from {self.url}/{endpoint}...")
        return json.dumps({"data": f"Response from {endpoint}"})

# Proxy local
class RemoteServiceProxy:
    def __init__(self, url: str):
        self.url = url
        self.cache = {}
        self.remote_service = None
    
    def _get_remote_service(self):
        if self.remote_service is None:
            self.remote_service = RemoteService(self.url)
        return self.remote_service
    
    def fetch_data(self, endpoint: str, use_cache: bool = True):
        if use_cache and endpoint in self.cache:
            print(f"Returning cached data for {endpoint}")
            return self.cache[endpoint]
        
        data = self._get_remote_service().fetch_data(endpoint)
        if use_cache:
            self.cache[endpoint] = data
        return data

# Utilisation
proxy = RemoteServiceProxy("https://api.example.com")
data1 = proxy.fetch_data("users")  # Appel réseau
data2 = proxy.fetch_data("users")  # Depuis le cache
```

### Cas d'usage concret : Cache de résultats coûteux

```python
import time
from functools import wraps

class ExpensiveOperation:
    def compute(self, n: int) -> int:
        """Opération coûteuse (simulée)"""
        print(f"Computing for {n}...")
        time.sleep(1)  # Simulation d'une opération longue
        return n * 2

class CachingProxy:
    def __init__(self, operation: ExpensiveOperation):
        self.operation = operation
        self.cache = {}
    
    def compute(self, n: int) -> int:
        if n in self.cache:
            print(f"Cache hit for {n}")
            return self.cache[n]
        
        result = self.operation.compute(n)
        self.cache[n] = result
        return result

# Utilisation
expensive_op = ExpensiveOperation()
cached_op = CachingProxy(expensive_op)

start = time.time()
result1 = cached_op.compute(5)  # Prend 1 seconde
print(f"Time: {time.time() - start:.2f}s")

start = time.time()
result2 = cached_op.compute(5)  # Instantané (cache)
print(f"Time: {time.time() - start:.2f}s")
```

### Quand utiliser Proxy ?

✅ **Utilisez-le quand** :
- Vous voulez un chargement paresseux (lazy loading)
- Vous avez besoin de contrôle d'accès
- Vous voulez mettre en cache des résultats
- Vous travaillez avec des objets distants

---

## 5. Composite

### Intention

Composer des objets en structures arborescentes pour représenter des hiérarchies partie-tout. Composite permet aux clients de traiter uniformément les objets individuels et les compositions d'objets.

### Problème résolu

Comment traiter uniformément des objets individuels et des groupes d'objets ?

### Solution

```python
from abc import ABC, abstractmethod

class FileSystemComponent(ABC):
    @abstractmethod
    def get_size(self) -> int:
        pass
    
    @abstractmethod
    def display(self, indent: str = ""):
        pass

class File(FileSystemComponent):
    def __init__(self, name: str, size: int):
        self.name = name
        self.size = size
    
    def get_size(self) -> int:
        return self.size
    
    def display(self, indent: str = ""):
        print(f"{indent}📄 {self.name} ({self.size} bytes)")

class Folder(FileSystemComponent):
    def __init__(self, name: str):
        self.name = name
        self.children = []
    
    def add(self, component: FileSystemComponent):
        self.children.append(component)
    
    def remove(self, component: FileSystemComponent):
        self.children.remove(component)
    
    def get_size(self) -> int:
        total = 0
        for child in self.children:
            total += child.get_size()
        return total
    
    def display(self, indent: str = ""):
        print(f"{indent}📁 {self.name} ({self.get_size()} bytes)")
        for child in self.children:
            child.display(indent + "  ")

# Utilisation
root = Folder("root")
documents = Folder("Documents")
photos = Folder("Photos")

file1 = File("readme.txt", 100)
file2 = File("notes.txt", 200)
photo1 = File("vacation.jpg", 5000)
photo2 = File("family.jpg", 4000)

documents.add(file1)
documents.add(file2)
photos.add(photo1)
photos.add(photo2)

root.add(documents)
root.add(photos)

root.display()
```

### Cas d'usage concret : Menu hiérarchique

```python
from abc import ABC, abstractmethod

class MenuComponent(ABC):
    @abstractmethod
    def render(self, indent: str = ""):
        pass
    
    @abstractmethod
    def get_url(self) -> str:
        pass

class MenuItem(MenuComponent):
    def __init__(self, name: str, url: str):
        self.name = name
        self.url = url
    
    def render(self, indent: str = ""):
        print(f"{indent}- {self.name} ({self.url})")
    
    def get_url(self) -> str:
        return self.url

class Menu(MenuComponent):
    def __init__(self, name: str):
        self.name = name
        self.children = []
    
    def add(self, component: MenuComponent):
        self.children.append(component)
    
    def render(self, indent: str = ""):
        print(f"{indent}📋 {self.name}")
        for child in self.children:
            child.render(indent + "  ")
    
    def get_url(self) -> str:
        # Retourne l'URL du premier élément ou "#"
        if self.children:
            return self.children[0].get_url()
        return "#"

# Utilisation
main_menu = Menu("Main Menu")
products_menu = Menu("Products")
services_menu = Menu("Services")

products_menu.add(MenuItem("Laptops", "/products/laptops"))
products_menu.add(MenuItem("Phones", "/products/phones"))
services_menu.add(MenuItem("Support", "/services/support"))
services_menu.add(MenuItem("Consulting", "/services/consulting"))

main_menu.add(MenuItem("Home", "/"))
main_menu.add(products_menu)
main_menu.add(services_menu)
main_menu.add(MenuItem("About", "/about"))

main_menu.render()
```

### Quand utiliser Composite ?

✅ **Utilisez-le quand** :
- Vous avez une structure hiérarchique d'objets
- Vous voulez traiter uniformément les objets individuels et les groupes
- Vous avez des structures arborescentes

---

## Résumé des Patterns Structurels

| Pattern | Problème résolu | Quand l'utiliser |
|---------|----------------|------------------|
| **Adapter** | Interfaces incompatibles | Intégration de bibliothèques tierces |
| **Decorator** | Ajouter des fonctionnalités dynamiquement | Validation, logging, caching |
| **Facade** | Simplifier un sous-système complexe | APIs simplifiées, wrappers |
| **Proxy** | Contrôler l'accès à un objet | Lazy loading, sécurité, cache |
| **Composite** | Structures hiérarchiques | Arborescences, menus, systèmes de fichiers |

---

**Prochaine étape** : Découvrez les [Patterns Comportementaux](./04-patterns-comportementaux.md) pour gérer les interactions entre objets.
