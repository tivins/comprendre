# Exemples Concrets

Ce document présente des exemples complets et réalistes d'application des design patterns dans des projets concrets. Chaque exemple montre comment plusieurs patterns peuvent travailler ensemble pour résoudre des problèmes réels.

## Exemple 1 : Système E-Commerce

### Contexte

Nous construisons un système e-commerce avec :
- Gestion des produits
- Panier d'achat
- Calcul des prix avec différentes remises
- Traitement des paiements
- Notifications aux clients

### Patterns utilisés

- **Factory Method** : Création des méthodes de paiement
- **Strategy** : Calcul des remises
- **Observer** : Notifications d'événements
- **Command** : Traitement des commandes avec undo/redo

### Implémentation

```python
from abc import ABC, abstractmethod
from typing import List, Dict
from datetime import datetime

# ========== FACTORY METHOD : Méthodes de paiement ==========

class PaymentMethod(ABC):
    @abstractmethod
    def pay(self, amount: float) -> bool:
        pass

class CreditCard(PaymentMethod):
    def __init__(self, card_number: str, cvv: str):
        self.card_number = card_number
        self.cvv = cvv
    
    def pay(self, amount: float) -> bool:
        print(f"Processing credit card payment: ${amount:.2f}")
        return True

class PayPal(PaymentMethod):
    def __init__(self, email: str):
        self.email = email
    
    def pay(self, amount: float) -> bool:
        print(f"Processing PayPal payment: ${amount:.2f} to {self.email}")
        return True

class PaymentMethodFactory:
    @staticmethod
    def create(method_type: str, **kwargs) -> PaymentMethod:
        if method_type == "credit_card":
            return CreditCard(kwargs["card_number"], kwargs["cvv"])
        elif method_type == "paypal":
            return PayPal(kwargs["email"])
        else:
            raise ValueError(f"Unknown payment method: {method_type}")

# ========== STRATEGY : Calcul des remises ==========

class DiscountStrategy(ABC):
    @abstractmethod
    def calculate(self, amount: float) -> float:
        pass

class NoDiscount(DiscountStrategy):
    def calculate(self, amount: float) -> float:
        return amount

class PercentageDiscount(DiscountStrategy):
    def __init__(self, percentage: float):
        self.percentage = percentage
    
    def calculate(self, amount: float) -> float:
        return amount * (1 - self.percentage / 100)

class CouponDiscount(DiscountStrategy):
    def __init__(self, discount_amount: float, min_purchase: float):
        self.discount_amount = discount_amount
        self.min_purchase = min_purchase
    
    def calculate(self, amount: float) -> float:
        if amount >= self.min_purchase:
            return max(0, amount - self.discount_amount)
        return amount

# ========== OBSERVER : Notifications ==========

class OrderObserver(ABC):
    @abstractmethod
    def notify(self, event: str, data: Dict):
        pass

class EmailNotifier(OrderObserver):
    def notify(self, event: str, data: Dict):
        if event == "order_placed":
            print(f"📧 Email: Order #{data['order_id']} placed successfully!")
        elif event == "order_shipped":
            print(f"📧 Email: Order #{data['order_id']} has been shipped!")

class SMSNotifier(OrderObserver):
    def notify(self, event: str, data: Dict):
        if event == "order_placed":
            print(f"📱 SMS: Your order #{data['order_id']} is confirmed!")

class InventoryManager(OrderObserver):
    def notify(self, event: str, data: Dict):
        if event == "order_placed":
            print(f"📦 Inventory: Updating stock for order #{data['order_id']}")

# ========== COMMAND : Traitement des commandes ==========

class OrderCommand(ABC):
    @abstractmethod
    def execute(self):
        pass
    
    @abstractmethod
    def undo(self):
        pass

class AddItemCommand(OrderCommand):
    def __init__(self, order: 'Order', product_id: str, quantity: int, price: float):
        self.order = order
        self.product_id = product_id
        self.quantity = quantity
        self.price = price
    
    def execute(self):
        self.order.add_item(self.product_id, self.quantity, self.price)
    
    def undo(self):
        self.order.remove_item(self.product_id)

class ApplyDiscountCommand(OrderCommand):
    def __init__(self, order: 'Order', discount: DiscountStrategy):
        self.order = order
        self.discount = discount
        self.previous_discount = None
    
    def execute(self):
        self.previous_discount = self.order.discount_strategy
        self.order.set_discount(self.discount)
    
    def undo(self):
        if self.previous_discount:
            self.order.set_discount(self.previous_discount)

# ========== CLASSE PRINCIPALE : Order ==========

class Order:
    def __init__(self):
        self.order_id = f"ORD-{datetime.now().strftime('%Y%m%d%H%M%S')}"
        self.items: Dict[str, Dict] = {}
        self.discount_strategy: DiscountStrategy = NoDiscount()
        self.observers: List[OrderObserver] = []
        self.command_history: List[OrderCommand] = []
    
    def attach_observer(self, observer: OrderObserver):
        self.observers.append(observer)
    
    def notify_observers(self, event: str, data: Dict):
        for observer in self.observers:
            observer.notify(event, data)
    
    def add_item(self, product_id: str, quantity: int, price: float):
        if product_id in self.items:
            self.items[product_id]["quantity"] += quantity
        else:
            self.items[product_id] = {"quantity": quantity, "price": price}
    
    def remove_item(self, product_id: str):
        if product_id in self.items:
            del self.items[product_id]
    
    def set_discount(self, discount: DiscountStrategy):
        self.discount_strategy = discount
    
    def calculate_subtotal(self) -> float:
        return sum(item["quantity"] * item["price"] for item in self.items.values())
    
    def calculate_total(self) -> float:
        subtotal = self.calculate_subtotal()
        return self.discount_strategy.calculate(subtotal)
    
    def execute_command(self, command: OrderCommand):
        command.execute()
        self.command_history.append(command)
    
    def undo_last_command(self):
        if self.command_history:
            command = self.command_history.pop()
            command.undo()
    
    def checkout(self, payment_method: PaymentMethod) -> bool:
        total = self.calculate_total()
        print(f"\n=== Checkout ===")
        print(f"Order ID: {self.order_id}")
        print(f"Items: {len(self.items)}")
        print(f"Subtotal: ${self.calculate_subtotal():.2f}")
        print(f"Discount applied")
        print(f"Total: ${total:.2f}")
        
        success = payment_method.pay(total)
        
        if success:
            self.notify_observers("order_placed", {
                "order_id": self.order_id,
                "total": total
            })
        
        return success

# ========== UTILISATION ==========

def main():
    # Créer une commande
    order = Order()
    
    # Attacher les observateurs
    order.attach_observer(EmailNotifier())
    order.attach_observer(SMSNotifier())
    order.attach_observer(InventoryManager())
    
    # Ajouter des items avec Command pattern
    order.execute_command(AddItemCommand(order, "LAPTOP001", 1, 999.99))
    order.execute_command(AddItemCommand(order, "MOUSE001", 2, 29.99))
    
    # Appliquer une remise avec Command pattern
    coupon = CouponDiscount(50.0, 1000.0)
    order.execute_command(ApplyDiscountCommand(order, coupon))
    
    # Créer une méthode de paiement avec Factory
    payment = PaymentMethodFactory.create(
        "credit_card",
        card_number="1234567890123456",
        cvv="123"
    )
    
    # Finaliser la commande
    order.checkout(payment)
    
    # Possibilité d'annuler
    print("\n--- Undo last action ---")
    order.undo_last_command()
    print(f"New total: ${order.calculate_total():.2f}")

if __name__ == "__main__":
    main()
```

---

## Exemple 2 : Framework de Validation

### Contexte

Création d'un système de validation flexible et extensible pour un formulaire d'inscription utilisateur.

### Patterns utilisés

- **Chain of Responsibility** : Validation en chaîne
- **Decorator** : Ajout de règles de validation
- **Strategy** : Différentes stratégies de validation

### Implémentation

```python
from abc import ABC, abstractmethod
from typing import Tuple, List

# ========== CHAIN OF RESPONSIBILITY : Validation ==========

class Validator(ABC):
    def __init__(self):
        self._next_validator = None
    
    def set_next(self, validator: 'Validator') -> 'Validator':
        self._next_validator = validator
        return validator
    
    @abstractmethod
    def validate(self, value: str) -> Tuple[bool, str]:
        if self._next_validator:
            return self._next_validator.validate(value)
        return True, ""

class RequiredValidator(Validator):
    def validate(self, value: str) -> Tuple[bool, str]:
        if not value or not value.strip():
            return False, "This field is required"
        return super().validate(value)

class MinLengthValidator(Validator):
    def __init__(self, min_length: int):
        super().__init__()
        self.min_length = min_length
    
    def validate(self, value: str) -> Tuple[bool, str]:
        if len(value) < self.min_length:
            return False, f"Minimum length is {self.min_length} characters"
        return super().validate(value)

class MaxLengthValidator(Validator):
    def __init__(self, max_length: int):
        super().__init__()
        self.max_length = max_length
    
    def validate(self, value: str) -> Tuple[bool, str]:
        if len(value) > self.max_length:
            return False, f"Maximum length is {self.max_length} characters"
        return super().validate(value)

class EmailFormatValidator(Validator):
    def validate(self, value: str) -> Tuple[bool, str]:
        if "@" not in value or "." not in value.split("@")[1]:
            return False, "Invalid email format"
        return super().validate(value)

class PasswordStrengthValidator(Validator):
    def validate(self, value: str) -> Tuple[bool, str]:
        has_upper = any(c.isupper() for c in value)
        has_lower = any(c.islower() for c in value)
        has_digit = any(c.isdigit() for c in value)
        has_special = any(c in "!@#$%^&*" for c in value)
        
        if not (has_upper and has_lower and has_digit and has_special):
            return False, "Password must contain uppercase, lowercase, digit, and special character"
        return super().validate(value)

# ========== DECORATOR : Ajout de fonctionnalités ==========

class ValidationDecorator(ABC):
    def __init__(self, validator: Validator):
        self._validator = validator
    
    @abstractmethod
    def validate(self, value: str) -> Tuple[bool, str]:
        pass

class TrimDecorator(ValidationDecorator):
    """Décore un validateur pour trimmer la valeur avant validation"""
    def validate(self, value: str) -> Tuple[bool, str]:
        trimmed_value = value.strip() if value else ""
        return self._validator.validate(trimmed_value)

class CaseInsensitiveDecorator(ValidationDecorator):
    """Décore un validateur pour ignorer la casse"""
    def validate(self, value: str) -> Tuple[bool, str]:
        lower_value = value.lower() if value else ""
        return self._validator.validate(lower_value)

# ========== STRATEGY : Différentes stratégies de validation ==========

class ValidationStrategy(ABC):
    @abstractmethod
    def validate_all(self, fields: dict) -> dict:
        pass

class StrictValidationStrategy(ValidationStrategy):
    """Arrête à la première erreur"""
    def __init__(self, validators: dict):
        self.validators = validators
    
    def validate_all(self, fields: dict) -> dict:
        errors = {}
        for field_name, value in fields.items():
            if field_name in self.validators:
                is_valid, error = self.validators[field_name].validate(value)
                if not is_valid:
                    errors[field_name] = error
                    break  # Arrête à la première erreur
        return errors

class CollectAllErrorsStrategy(ValidationStrategy):
    """Collecte toutes les erreurs"""
    def __init__(self, validators: dict):
        self.validators = validators
    
    def validate_all(self, fields: dict) -> dict:
        errors = {}
        for field_name, value in fields.items():
            if field_name in self.validators:
                is_valid, error = self.validators[field_name].validate(value)
                if not is_valid:
                    errors[field_name] = error
        return errors

# ========== FORMULAIRE D'INSCRIPTION ==========

class RegistrationForm:
    def __init__(self, validation_strategy: ValidationStrategy):
        self.validation_strategy = validation_strategy
        self.fields = {}
    
    def set_field(self, name: str, value: str):
        self.fields[name] = value
    
    def validate(self) -> dict:
        return self.validation_strategy.validate_all(self.fields)
    
    def is_valid(self) -> bool:
        return len(self.validate()) == 0

# ========== UTILISATION ==========

def main():
    # Construire les validateurs avec Chain of Responsibility
    username_validator = RequiredValidator()
    username_validator.set_next(MinLengthValidator(3)).set_next(MaxLengthValidator(20))
    
    email_validator = RequiredValidator()
    email_validator.set_next(EmailFormatValidator())
    
    password_validator = RequiredValidator()
    password_validator.set_next(MinLengthValidator(8)).set_next(PasswordStrengthValidator())
    
    # Utiliser des décorateurs
    email_validator = TrimDecorator(email_validator)
    
    validators = {
        "username": username_validator,
        "email": email_validator,
        "password": password_validator
    }
    
    # Stratégie : collecter toutes les erreurs
    strategy = CollectAllErrorsStrategy(validators)
    form = RegistrationForm(strategy)
    
    # Test avec des données invalides
    form.set_field("username", "ab")  # Trop court
    form.set_field("email", "  invalid-email  ")  # Format invalide (sera trimé)
    form.set_field("password", "weak")  # Trop faible
    
    errors = form.validate()
    print("=== Validation Errors ===")
    for field, error in errors.items():
        print(f"{field}: {error}")
    
    # Test avec des données valides
    print("\n=== Valid Data ===")
    form.set_field("username", "johndoe")
    form.set_field("email", "john@example.com")
    form.set_field("password", "StrongP@ss123")
    
    if form.is_valid():
        print("✓ All fields are valid!")
    else:
        errors = form.validate()
        for field, error in errors.items():
            print(f"{field}: {error}")

if __name__ == "__main__":
    main()
```

---

## Exemple 3 : Système de Cache avec Proxy

### Contexte

Création d'un système de cache intelligent pour des appels API coûteux avec expiration automatique.

### Patterns utilisés

- **Proxy** : Cache avec lazy loading
- **Strategy** : Différentes stratégies de cache (LRU, FIFO, etc.)
- **Observer** : Notification d'expiration du cache

### Implémentation

```python
from abc import ABC, abstractmethod
from typing import Dict, Optional, Callable
from datetime import datetime, timedelta
from collections import OrderedDict

# ========== PROXY : Cache avec lazy loading ==========

class DataService(ABC):
    @abstractmethod
    def fetch(self, key: str) -> str:
        pass

class ExpensiveDataService(DataService):
    """Service coûteux simulant un appel API"""
    def __init__(self):
        self.call_count = 0
    
    def fetch(self, key: str) -> str:
        self.call_count += 1
        print(f"[API Call #{self.call_count}] Fetching data for key: {key}")
        # Simulation d'un appel réseau coûteux
        import time
        time.sleep(0.1)
        return f"Data for {key}"

class CacheEntry:
    def __init__(self, value: str, ttl: timedelta):
        self.value = value
        self.created_at = datetime.now()
        self.ttl = ttl
    
    def is_expired(self) -> bool:
        return datetime.now() - self.created_at > self.ttl

class CacheProxy(DataService):
    def __init__(self, service: DataService, ttl: timedelta = timedelta(minutes=5)):
        self.service = service
        self.cache: Dict[str, CacheEntry] = {}
        self.ttl = ttl
        self.hits = 0
        self.misses = 0
    
    def fetch(self, key: str) -> str:
        # Vérifier le cache
        if key in self.cache:
            entry = self.cache[key]
            if not entry.is_expired():
                self.hits += 1
                print(f"[Cache HIT] Returning cached data for: {key}")
                return entry.value
            else:
                # Expiré, le retirer
                del self.cache[key]
        
        # Cache miss, appeler le service réel
        self.misses += 1
        print(f"[Cache MISS] Fetching from service for: {key}")
        value = self.service.fetch(key)
        
        # Mettre en cache
        self.cache[key] = CacheEntry(value, self.ttl)
        return value
    
    def clear(self):
        self.cache.clear()
        print("Cache cleared")
    
    def get_stats(self) -> dict:
        total = self.hits + self.misses
        hit_rate = (self.hits / total * 100) if total > 0 else 0
        return {
            "hits": self.hits,
            "misses": self.misses,
            "hit_rate": f"{hit_rate:.1f}%",
            "cached_items": len(self.cache)
        }

# ========== STRATEGY : Stratégies de remplacement de cache ==========

class EvictionStrategy(ABC):
    @abstractmethod
    def evict(self, cache: Dict) -> str:
        pass

class LRUStrategy(EvictionStrategy):
    """Least Recently Used"""
    def evict(self, cache: OrderedDict) -> str:
        # Le premier élément est le moins récemment utilisé
        key = next(iter(cache))
        del cache[key]
        return key

class FIFOStrategy(EvictionStrategy):
    """First In First Out"""
    def evict(self, cache: OrderedDict) -> str:
        # Similaire à LRU pour OrderedDict
        key = next(iter(cache))
        del cache[key]
        return key

class LRUCacheProxy(CacheProxy):
    def __init__(self, service: DataService, max_size: int = 10, 
                 ttl: timedelta = timedelta(minutes=5),
                 eviction_strategy: EvictionStrategy = None):
        super().__init__(service, ttl)
        self.max_size = max_size
        self.cache = OrderedDict()  # Pour maintenir l'ordre
        self.eviction_strategy = eviction_strategy or LRUStrategy()
    
    def fetch(self, key: str) -> str:
        # Si la clé existe, la déplacer à la fin (most recently used)
        if key in self.cache:
            entry = self.cache[key]
            if not entry.is_expired():
                self.cache.move_to_end(key)  # Marquer comme récemment utilisé
                self.hits += 1
                print(f"[LRU Cache HIT] {key}")
                return entry.value
            else:
                del self.cache[key]
        
        # Si le cache est plein, évincer
        if len(self.cache) >= self.max_size:
            evicted = self.eviction_strategy.evict(self.cache)
            print(f"[LRU Cache] Evicted: {evicted}")
        
        # Cache miss
        self.misses += 1
        value = self.service.fetch(key)
        self.cache[key] = CacheEntry(value, self.ttl)
        self.cache.move_to_end(key)  # Ajouter à la fin
        return value

# ========== UTILISATION ==========

def main():
    print("=== Simple Cache Proxy ===")
    service = ExpensiveDataService()
    cache = CacheProxy(service, ttl=timedelta(seconds=2))
    
    print("\nFirst call (cache miss):")
    cache.fetch("user_123")
    
    print("\nSecond call (cache hit):")
    cache.fetch("user_123")
    
    print("\nAfter expiration (cache miss):")
    import time
    time.sleep(2.1)
    cache.fetch("user_123")
    
    print(f"\nCache stats: {cache.get_stats()}")
    
    print("\n=== LRU Cache Proxy ===")
    lru_cache = LRUCacheProxy(service, max_size=3)
    
    print("\nFilling cache:")
    lru_cache.fetch("A")
    lru_cache.fetch("B")
    lru_cache.fetch("C")
    
    print("\nAccessing A (moves to end):")
    lru_cache.fetch("A")
    
    print("\nAdding D (evicts B, least recently used):")
    lru_cache.fetch("D")
    
    print(f"\nLRU Cache stats: {lru_cache.get_stats()}")

if __name__ == "__main__":
    main()
```

---

## Exemple 4 : Application de Gestion de Tâches

### Contexte

Application de gestion de tâches avec :
- États de tâches (todo, in_progress, done)
- Filtres et recherches
- Historique avec undo/redo
- Export dans différents formats

### Patterns utilisés

- **State** : États des tâches
- **Command** : Undo/redo
- **Strategy** : Export dans différents formats
- **Composite** : Groupes de tâches

### Implémentation

```python
from abc import ABC, abstractmethod
from typing import List, Optional
from datetime import datetime
from enum import Enum

# ========== STATE : États des tâches ==========

class TaskState(ABC):
    @abstractmethod
    def start(self, task: 'Task'):
        pass
    
    @abstractmethod
    def complete(self, task: 'Task'):
        pass
    
    @abstractmethod
    def cancel(self, task: 'Task'):
        pass

class TodoState(TaskState):
    def start(self, task: 'Task'):
        print(f"Starting task: {task.title}")
        task.set_state(InProgressState())
    
    def complete(self, task: 'Task'):
        print("Cannot complete: task not started")
    
    def cancel(self, task: 'Task'):
        print(f"Cancelling task: {task.title}")
        task.set_state(CancelledState())

class InProgressState(TaskState):
    def start(self, task: 'Task'):
        print("Task already in progress")
    
    def complete(self, task: 'Task'):
        print(f"Completing task: {task.title}")
        task.set_state(DoneState())
    
    def cancel(self, task: 'Task'):
        print(f"Cancelling task: {task.title}")
        task.set_state(CancelledState())

class DoneState(TaskState):
    def start(self, task: 'Task'):
        print("Cannot start: task already completed")
    
    def complete(self, task: 'Task'):
        print("Task already completed")
    
    def cancel(self, task: 'Task'):
        print("Cannot cancel: task already completed")

class CancelledState(TaskState):
    def start(self, task: 'Task'):
        print("Cannot start: task is cancelled")
    
    def complete(self, task: 'Task'):
        print("Cannot complete: task is cancelled")
    
    def cancel(self, task: 'Task'):
        print("Task already cancelled")

# ========== COMMAND : Undo/Redo ==========

class TaskCommand(ABC):
    @abstractmethod
    def execute(self):
        pass
    
    @abstractmethod
    def undo(self):
        pass

class ChangeStateCommand(TaskCommand):
    def __init__(self, task: 'Task', new_state: TaskState):
        self.task = task
        self.new_state = new_state
        self.previous_state = None
    
    def execute(self):
        self.previous_state = self.task.state
        self.task.set_state(self.new_state)
    
    def undo(self):
        if self.previous_state:
            self.task.set_state(self.previous_state)

class UpdateTitleCommand(TaskCommand):
    def __init__(self, task: 'Task', new_title: str):
        self.task = task
        self.new_title = new_title
        self.previous_title = None
    
    def execute(self):
        self.previous_title = self.task.title
        self.task.title = self.new_title
    
    def undo(self):
        if self.previous_title:
            self.task.title = self.previous_title

# ========== STRATEGY : Export ==========

class ExportStrategy(ABC):
    @abstractmethod
    def export(self, tasks: List['Task']) -> str:
        pass

class JSONExportStrategy(ExportStrategy):
    def export(self, tasks: List['Task']) -> str:
        import json
        data = [{"title": t.title, "state": type(t.state).__name__} for t in tasks]
        return json.dumps(data, indent=2)

class CSVExportStrategy(ExportStrategy):
    def export(self, tasks: List['Task']) -> str:
        lines = ["Title,State"]
        for task in tasks:
            lines.append(f"{task.title},{type(task.state).__name__}")
        return "\n".join(lines)

class MarkdownExportStrategy(ExportStrategy):
    def export(self, tasks: List['Task']) -> str:
        lines = ["# Tasks\n"]
        for task in tasks:
            lines.append(f"- [{type(task.state).__name__}] {task.title}")
        return "\n".join(lines)

# ========== COMPOSITE : Groupes de tâches ==========

class TaskComponent(ABC):
    @abstractmethod
    def get_title(self) -> str:
        pass
    
    @abstractmethod
    def get_state(self) -> str:
        pass

class Task(TaskComponent):
    def __init__(self, title: str):
        self.title = title
        self.state: TaskState = TodoState()
        self.created_at = datetime.now()
    
    def set_state(self, state: TaskState):
        self.state = state
    
    def start(self):
        self.state.start(self)
    
    def complete(self):
        self.state.complete(self)
    
    def cancel(self):
        self.state.cancel(self)
    
    def get_title(self) -> str:
        return self.title
    
    def get_state(self) -> str:
        return type(self.state).__name__

class TaskGroup(TaskComponent):
    def __init__(self, name: str):
        self.name = name
        self.children: List[TaskComponent] = []
    
    def add(self, component: TaskComponent):
        self.children.append(component)
    
    def get_title(self) -> str:
        return self.name
    
    def get_state(self) -> str:
        states = [child.get_state() for child in self.children]
        if all(s == "DoneState" for s in states):
            return "DoneState"
        elif any(s == "InProgressState" for s in states):
            return "InProgressState"
        return "TodoState"
    
    def get_all_tasks(self) -> List[Task]:
        tasks = []
        for child in self.children:
            if isinstance(child, Task):
                tasks.append(child)
            elif isinstance(child, TaskGroup):
                tasks.extend(child.get_all_tasks())
        return tasks

# ========== APPLICATION ==========

class TaskManager:
    def __init__(self):
        self.tasks: List[Task] = []
        self.history: List[TaskCommand] = []
        self.redo_stack: List[TaskCommand] = []
    
    def add_task(self, title: str) -> Task:
        task = Task(title)
        self.tasks.append(task)
        return task
    
    def execute_command(self, command: TaskCommand):
        command.execute()
        self.history.append(command)
        self.redo_stack.clear()
    
    def undo(self):
        if self.history:
            command = self.history.pop()
            command.undo()
            self.redo_stack.append(command)
    
    def redo(self):
        if self.redo_stack:
            command = self.redo_stack.pop()
            command.execute()
            self.history.append(command)
    
    def export(self, strategy: ExportStrategy) -> str:
        return strategy.export(self.tasks)

# ========== UTILISATION ==========

def main():
    manager = TaskManager()
    
    # Créer des tâches
    task1 = manager.add_task("Implement feature X")
    task2 = manager.add_task("Write tests")
    task3 = manager.add_task("Update documentation")
    
    # Utiliser Command pour changer d'état
    manager.execute_command(ChangeStateCommand(task1, InProgressState()))
    manager.execute_command(ChangeStateCommand(task1, DoneState()))
    
    # Modifier le titre avec Command
    manager.execute_command(UpdateTitleCommand(task2, "Write unit tests"))
    
    # Undo
    print("\n=== Undo ===")
    manager.undo()  # Annule le changement de titre
    print(f"Task2 title: {task2.title}")
    
    manager.undo()  # Annule le changement d'état
    print(f"Task1 state: {task1.get_state()}")
    
    # Redo
    print("\n=== Redo ===")
    manager.redo()
    print(f"Task1 state: {task1.get_state()}")
    
    # Export avec différentes stratégies
    print("\n=== Export JSON ===")
    print(manager.export(JSONExportStrategy()))
    
    print("\n=== Export CSV ===")
    print(manager.export(CSVExportStrategy()))
    
    print("\n=== Export Markdown ===")
    print(manager.export(MarkdownExportStrategy()))
    
    # Composite : Groupes de tâches
    print("\n=== Task Groups ===")
    sprint1 = TaskGroup("Sprint 1")
    sprint1.add(task1)
    sprint1.add(task2)
    
    print(f"Sprint 1 state: {sprint1.get_state()}")

if __name__ == "__main__":
    main()
```

---

## Résumé

Ces exemples montrent comment :

1. **Combiner plusieurs patterns** pour résoudre des problèmes complexes
2. **Adapter les patterns** aux besoins spécifiques de votre projet
3. **Créer des systèmes flexibles** et maintenables
4. **Réutiliser du code** grâce aux abstractions

Les patterns ne sont pas des solutions isolées - ils fonctionnent mieux ensemble pour créer des architectures robustes et évolutives.

---

**Prochaine étape** : Consultez le guide de [Mise en Pratique](./06-mise-en-pratique.md) pour apprendre à choisir et appliquer les patterns dans vos projets.
