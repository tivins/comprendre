# Patterns Comportementaux (Behavioral Patterns)

Les patterns comportementaux se concentrent sur la communication entre objets et la répartition des responsabilités. Ils définissent comment les objets interagissent et se partagent les responsabilités.

## 1. Observer

### Intention

Définir une dépendance un-à-plusieurs entre objets de sorte que lorsqu'un objet change d'état, tous ses dépendants sont notifiés et mis à jour automatiquement.

### Problème résolu

Comment être notifié quand un événement se produit sans créer un couplage fort ?

### Solution

```python
from abc import ABC, abstractmethod

class Observer(ABC):
    @abstractmethod
    def update(self, message: str):
        pass

class Subject(ABC):
    def __init__(self):
        self._observers = []
    
    def attach(self, observer: Observer):
        self._observers.append(observer)
    
    def detach(self, observer: Observer):
        self._observers.remove(observer)
    
    def notify(self, message: str):
        for observer in self._observers:
            observer.update(message)

# Sujet concret
class NewsAgency(Subject):
    def __init__(self):
        super().__init__()
        self._news = None
    
    def set_news(self, news: str):
        self._news = news
        self.notify(news)

# Observateurs concrets
class EmailSubscriber(Observer):
    def __init__(self, email: str):
        self.email = email
    
    def update(self, message: str):
        print(f"Email to {self.email}: {message}")

class SMSSubscriber(Observer):
    def __init__(self, phone: str):
        self.phone = phone
    
    def update(self, message: str):
        print(f"SMS to {self.phone}: {message}")

class PushSubscriber(Observer):
    def __init__(self, device_id: str):
        self.device_id = device_id
    
    def update(self, message: str):
        print(f"Push to {self.device_id}: {message}")

# Utilisation
agency = NewsAgency()

email_sub = EmailSubscriber("user@example.com")
sms_sub = SMSSubscriber("+33123456789")
push_sub = PushSubscriber("device_123")

agency.attach(email_sub)
agency.attach(sms_sub)
agency.attach(push_sub)

agency.set_news("Breaking: New product launched!")
# Tous les observateurs sont notifiés automatiquement
```

### Cas d'usage concret : Système de stock

```python
from abc import ABC, abstractmethod

class StockObserver(ABC):
    @abstractmethod
    def update(self, symbol: str, price: float):
        pass

class StockMarket:
    def __init__(self):
        self._observers = []
        self._prices = {}
    
    def attach(self, observer: StockObserver):
        self._observers.append(observer)
    
    def detach(self, observer: StockObserver):
        self._observers.remove(observer)
    
    def set_price(self, symbol: str, price: float):
        old_price = self._prices.get(symbol)
        self._prices[symbol] = price
        
        if old_price != price:
            self.notify(symbol, price)
    
    def notify(self, symbol: str, price: float):
        for observer in self._observers:
            observer.update(symbol, price)

class Trader(StockObserver):
    def __init__(self, name: str, symbol: str, threshold: float):
        self.name = name
        self.symbol = symbol
        self.threshold = threshold
    
    def update(self, symbol: str, price: float):
        if symbol == self.symbol and price >= self.threshold:
            print(f"{self.name}: {symbol} reached {price}! Time to sell!")

class Portfolio(StockObserver):
    def __init__(self, name: str):
        self.name = name
        self.holdings = {}
    
    def add_stock(self, symbol: str, quantity: int):
        self.holdings[symbol] = quantity
    
    def update(self, symbol: str, price: float):
        if symbol in self.holdings:
            value = self.holdings[symbol] * price
            print(f"{self.name}: {symbol} value is now ${value:.2f}")

# Utilisation
market = StockMarket()

trader1 = Trader("Alice", "AAPL", 150.0)
trader2 = Trader("Bob", "GOOGL", 2500.0)
portfolio = Portfolio("My Portfolio")
portfolio.add_stock("AAPL", 10)
portfolio.add_stock("GOOGL", 5)

market.attach(trader1)
market.attach(trader2)
market.attach(portfolio)

market.set_price("AAPL", 145.0)  # Pas de notification
market.set_price("AAPL", 150.0)  # Alice est notifiée
market.set_price("GOOGL", 2500.0)  # Bob est notifié
```

### Version avec événements typés (Python moderne)

```python
from typing import Callable, Dict, List
from dataclasses import dataclass

@dataclass
class Event:
    event_type: str
    data: dict

class EventEmitter:
    def __init__(self):
        self._listeners: Dict[str, List[Callable]] = {}
    
    def on(self, event_type: str, callback: Callable):
        if event_type not in self._listeners:
            self._listeners[event_type] = []
        self._listeners[event_type].append(callback)
    
    def off(self, event_type: str, callback: Callable):
        if event_type in self._listeners:
            self._listeners[event_type].remove(callback)
    
    def emit(self, event: Event):
        if event.event_type in self._listeners:
            for callback in self._listeners[event.event_type]:
                callback(event)

# Utilisation
emitter = EventEmitter()

emitter.on("user:created", lambda e: print(f"User created: {e.data['name']}"))
emitter.on("user:created", lambda e: print(f"Sending welcome email to {e.data['email']}"))
emitter.on("order:placed", lambda e: print(f"Order placed: ${e.data['amount']}"))

emitter.emit(Event("user:created", {"name": "John", "email": "john@example.com"}))
emitter.emit(Event("order:placed", {"amount": 100.0}))
```

### Quand utiliser Observer ?

✅ **Utilisez-le quand** :
- Un changement d'état nécessite de mettre à jour plusieurs objets
- Vous voulez découpler l'émetteur et les récepteurs
- Vous avez besoin d'un système d'événements

---

## 2. Strategy

### Intention

Définir une famille d'algorithmes, les encapsuler et les rendre interchangeables. Strategy permet à l'algorithme de varier indépendamment des clients qui l'utilisent.

### Problème résolu

Comment choisir un algorithme à l'exécution sans utiliser de conditions multiples ?

### Solution

```python
from abc import ABC, abstractmethod

class PaymentStrategy(ABC):
    @abstractmethod
    def pay(self, amount: float) -> bool:
        pass

class CreditCardStrategy(PaymentStrategy):
    def __init__(self, card_number: str, cvv: str):
        self.card_number = card_number
        self.cvv = cvv
    
    def pay(self, amount: float) -> bool:
        print(f"Paying {amount}€ with credit card ending in {self.card_number[-4:]}")
        return True

class PayPalStrategy(PaymentStrategy):
    def __init__(self, email: str):
        self.email = email
    
    def pay(self, amount: float) -> bool:
        print(f"Paying {amount}€ with PayPal ({self.email})")
        return True

class BankTransferStrategy(PaymentStrategy):
    def __init__(self, account_number: str):
        self.account_number = account_number
    
    def pay(self, amount: float) -> bool:
        print(f"Paying {amount}€ via bank transfer ({self.account_number})")
        return True

class ShoppingCart:
    def __init__(self):
        self.items = []
    
    def add_item(self, item: str, price: float):
        self.items.append((item, price))
    
    def checkout(self, payment_strategy: PaymentStrategy):
        total = sum(price for _, price in self.items)
        print(f"Total: {total}€")
        return payment_strategy.pay(total)

# Utilisation
cart = ShoppingCart()
cart.add_item("Laptop", 999.0)
cart.add_item("Mouse", 29.0)

# Choisir la stratégie à l'exécution
payment_method = input("Payment method (card/paypal/bank): ")

if payment_method == "card":
    strategy = CreditCardStrategy("1234567890123456", "123")
elif payment_method == "paypal":
    strategy = PayPalStrategy("user@example.com")
else:
    strategy = BankTransferStrategy("FR123456789")

cart.checkout(strategy)
```

### Cas d'usage concret : Calcul de remises

```python
from abc import ABC, abstractmethod

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

class FixedDiscount(DiscountStrategy):
    def __init__(self, discount_amount: float):
        self.discount_amount = discount_amount
    
    def calculate(self, amount: float) -> float:
        return max(0, amount - self.discount_amount)

class BuyOneGetOneFree(DiscountStrategy):
    def calculate(self, amount: float) -> float:
        # Simplifié : pour chaque 2 items, un est gratuit
        return amount / 2

class Order:
    def __init__(self, discount_strategy: DiscountStrategy):
        self.items = []
        self.discount_strategy = discount_strategy
    
    def add_item(self, name: str, price: float):
        self.items.append((name, price))
    
    def calculate_total(self) -> float:
        subtotal = sum(price for _, price in self.items)
        return self.discount_strategy.calculate(subtotal)
    
    def set_discount_strategy(self, strategy: DiscountStrategy):
        self.discount_strategy = strategy

# Utilisation
order = Order(NoDiscount())
order.add_item("Product 1", 50.0)
order.add_item("Product 2", 30.0)
print(f"Total (no discount): {order.calculate_total()}€")

order.set_discount_strategy(PercentageDiscount(10))
print(f"Total (10% off): {order.calculate_total()}€")

order.set_discount_strategy(FixedDiscount(20))
print(f"Total ($20 off): {order.calculate_total()}€")
```

### Cas d'usage concret : Tri de données

```python
from abc import ABC, abstractmethod
from typing import List

class SortStrategy(ABC):
    @abstractmethod
    def sort(self, data: List) -> List:
        pass

class BubbleSort(SortStrategy):
    def sort(self, data: List) -> List:
        arr = data.copy()
        n = len(arr)
        for i in range(n):
            for j in range(0, n - i - 1):
                if arr[j] > arr[j + 1]:
                    arr[j], arr[j + 1] = arr[j + 1], arr[j]
        return arr

class QuickSort(SortStrategy):
    def sort(self, data: List) -> List:
        if len(data) <= 1:
            return data
        pivot = data[len(data) // 2]
        left = [x for x in data if x < pivot]
        middle = [x for x in data if x == pivot]
        right = [x for x in data if x > pivot]
        return self.sort(left) + middle + self.sort(right)

class Sorter:
    def __init__(self, strategy: SortStrategy):
        self.strategy = strategy
    
    def set_strategy(self, strategy: SortStrategy):
        self.strategy = strategy
    
    def sort(self, data: List) -> List:
        return self.strategy.sort(data)

# Utilisation
data = [64, 34, 25, 12, 22, 11, 90]

sorter = Sorter(BubbleSort())
sorted_data = sorter.sort(data)
print(f"Bubble sort: {sorted_data}")

sorter.set_strategy(QuickSort())
sorted_data = sorter.sort(data)
print(f"Quick sort: {sorted_data}")
```

### Quand utiliser Strategy ?

✅ **Utilisez-le quand** :
- Vous avez plusieurs algorithmes interchangeables
- Vous voulez éviter des conditions multiples (if/else)
- Vous voulez isoler la logique algorithmique du code métier

---

## 3. Command

### Intention

Encapsuler une requête comme un objet, permettant de paramétrer les clients avec différentes requêtes, de mettre en file d'attente les requêtes et de prendre en charge les opérations d'annulation.

### Problème résolu

Comment découpler l'objet qui invoque une opération de l'objet qui l'exécute ?

### Solution

```python
from abc import ABC, abstractmethod

class Command(ABC):
    @abstractmethod
    def execute(self):
        pass
    
    @abstractmethod
    def undo(self):
        pass

class Light:
    def __init__(self, location: str):
        self.location = location
        self.is_on = False
    
    def turn_on(self):
        self.is_on = True
        print(f"{self.location} light is ON")
    
    def turn_off(self):
        self.is_on = False
        print(f"{self.location} light is OFF")

class LightOnCommand(Command):
    def __init__(self, light: Light):
        self.light = light
    
    def execute(self):
        self.light.turn_on()
    
    def undo(self):
        self.light.turn_off()

class LightOffCommand(Command):
    def __init__(self, light: Light):
        self.light = light
    
    def execute(self):
        self.light.turn_off()
    
    def undo(self):
        self.light.turn_on()

class RemoteControl:
    def __init__(self):
        self.commands = {}
        self.history = []
    
    def set_command(self, slot: int, command: Command):
        self.commands[slot] = command
    
    def press_button(self, slot: int):
        if slot in self.commands:
            command = self.commands[slot]
            command.execute()
            self.history.append(command)
    
    def undo(self):
        if self.history:
            command = self.history.pop()
            command.undo()

# Utilisation
living_room_light = Light("Living Room")
kitchen_light = Light("Kitchen")

remote = RemoteControl()
remote.set_command(0, LightOnCommand(living_room_light))
remote.set_command(1, LightOffCommand(living_room_light))
remote.set_command(2, LightOnCommand(kitchen_light))

remote.press_button(0)  # Allume le salon
remote.press_button(2)  # Allume la cuisine
remote.undo()  # Éteint la cuisine
remote.undo()  # Éteint le salon
```

### Cas d'usage concret : Éditeur de texte

```python
from abc import ABC, abstractmethod

class Document:
    def __init__(self):
        self.content = ""
        self.cursor = 0
    
    def insert(self, text: str, position: int):
        self.content = self.content[:position] + text + self.content[position:]
        self.cursor = position + len(text)
    
    def delete(self, position: int, length: int):
        deleted = self.content[position:position + length]
        self.content = self.content[:position] + self.content[position + length:]
        return deleted
    
    def get_content(self) -> str:
        return self.content

class Command(ABC):
    @abstractmethod
    def execute(self):
        pass
    
    @abstractmethod
    def undo(self):
        pass

class InsertCommand(Command):
    def __init__(self, document: Document, text: str, position: int):
        self.document = document
        self.text = text
        self.position = position
    
    def execute(self):
        self.document.insert(self.text, self.position)
    
    def undo(self):
        self.document.delete(self.position, len(self.text))

class DeleteCommand(Command):
    def __init__(self, document: Document, position: int, length: int):
        self.document = document
        self.position = position
        self.length = length
        self.deleted_text = None
    
    def execute(self):
        self.deleted_text = self.document.delete(self.position, self.length)
    
    def undo(self):
        if self.deleted_text:
            self.document.insert(self.deleted_text, self.position)

class TextEditor:
    def __init__(self):
        self.document = Document()
        self.history = []
        self.redo_stack = []
    
    def execute_command(self, command: Command):
        command.execute()
        self.history.append(command)
        self.redo_stack.clear()  # Effacer le redo quand on fait une nouvelle action
    
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
    
    def get_content(self) -> str:
        return self.document.get_content()

# Utilisation
editor = TextEditor()

editor.execute_command(InsertCommand(editor.document, "Hello", 0))
editor.execute_command(InsertCommand(editor.document, " World", 5))
print(editor.get_content())  # "Hello World"

editor.undo()
print(editor.get_content())  # "Hello"

editor.redo()
print(editor.get_content())  # "Hello World"
```

### Cas d'usage concret : File d'attente de tâches

```python
from abc import ABC, abstractmethod
from queue import Queue
import threading

class Task(ABC):
    @abstractmethod
    def execute(self):
        pass

class EmailTask(Task):
    def __init__(self, to: str, subject: str, body: str):
        self.to = to
        self.subject = subject
        self.body = body
    
    def execute(self):
        print(f"Sending email to {self.to}: {self.subject}")

class DatabaseTask(Task):
    def __init__(self, query: str):
        self.query = query
    
    def execute(self):
        print(f"Executing query: {self.query}")

class TaskQueue:
    def __init__(self):
        self.queue = Queue()
        self.worker_thread = None
        self.running = False
    
    def add_task(self, task: Task):
        self.queue.put(task)
    
    def start(self):
        self.running = True
        self.worker_thread = threading.Thread(target=self._worker)
        self.worker_thread.start()
    
    def stop(self):
        self.running = False
        if self.worker_thread:
            self.worker_thread.join()
    
    def _worker(self):
        while self.running:
            try:
                task = self.queue.get(timeout=1)
                task.execute()
                self.queue.task_done()
            except:
                continue

# Utilisation
queue = TaskQueue()
queue.start()

queue.add_task(EmailTask("user@example.com", "Welcome", "Hello!"))
queue.add_task(DatabaseTask("SELECT * FROM users"))
queue.add_task(EmailTask("admin@example.com", "Report", "Daily report"))

queue.stop()
```

### Quand utiliser Command ?

✅ **Utilisez-le quand** :
- Vous voulez paramétrer des objets avec des actions
- Vous avez besoin d'annulation/refaire (undo/redo)
- Vous voulez mettre en file d'attente des opérations
- Vous voulez découpler l'invocateur de l'exécuteur

---

## 4. Chain of Responsibility

### Intention

Éviter le couplage entre l'émetteur d'une requête et ses récepteurs en donnant à plusieurs objets la possibilité de traiter la requête. Chaînez les objets récepteurs et passez la requête le long de la chaîne jusqu'à ce qu'un objet la traite.

### Problème résolu

Comment traiter une requête avec plusieurs handlers possibles sans créer de couplage ?

### Solution

```python
from abc import ABC, abstractmethod

class Handler(ABC):
    def __init__(self):
        self._next_handler = None
    
    def set_next(self, handler: 'Handler') -> 'Handler':
        self._next_handler = handler
        return handler
    
    @abstractmethod
    def handle(self, request: str) -> str:
        if self._next_handler:
            return self._next_handler.handle(request)
        return None

class MonkeyHandler(Handler):
    def handle(self, request: str) -> str:
        if request == "Banana":
            return f"Monkey: I'll eat the {request}"
        return super().handle(request)

class SquirrelHandler(Handler):
    def handle(self, request: str) -> str:
        if request == "Nut":
            return f"Squirrel: I'll eat the {request}"
        return super().handle(request)

class DogHandler(Handler):
    def handle(self, request: str) -> str:
        if request == "MeatBall":
            return f"Dog: I'll eat the {request}"
        return super().handle(request)

# Utilisation
monkey = MonkeyHandler()
squirrel = SquirrelHandler()
dog = DogHandler()

monkey.set_next(squirrel).set_next(dog)

def client_code(handler: Handler):
    for food in ["Nut", "Banana", "Cup of coffee"]:
        print(f"\nClient: Who wants a {food}?")
        result = handler.handle(food)
        if result:
            print(f"  {result}")
        else:
            print(f"  {food} was left untouched.")

client_code(monkey)
```

### Cas d'usage concret : Validation en chaîne

```python
from abc import ABC, abstractmethod

class Validator(ABC):
    def __init__(self):
        self._next_validator = None
    
    def set_next(self, validator: 'Validator') -> 'Validator':
        self._next_validator = validator
        return validator
    
    @abstractmethod
    def validate(self, value: str) -> tuple[bool, str]:
        if self._next_validator:
            return self._next_validator.validate(value)
        return True, ""

class RequiredValidator(Validator):
    def validate(self, value: str) -> tuple[bool, str]:
        if not value:
            return False, "Field is required"
        return super().validate(value)

class MinLengthValidator(Validator):
    def __init__(self, min_length: int):
        super().__init__()
        self.min_length = min_length
    
    def validate(self, value: str) -> tuple[bool, str]:
        if len(value) < self.min_length:
            return False, f"Minimum length is {self.min_length}"
        return super().validate(value)

class EmailFormatValidator(Validator):
    def validate(self, value: str) -> tuple[bool, str]:
        if "@" not in value or "." not in value:
            return False, "Invalid email format"
        return super().validate(value)

class NoSpacesValidator(Validator):
    def validate(self, value: str) -> tuple[bool, str]:
        if " " in value:
            return False, "Email cannot contain spaces"
        return super().validate(value)

# Utilisation
email_validator = RequiredValidator()
email_validator.set_next(MinLengthValidator(5)).set_next(EmailFormatValidator()).set_next(NoSpacesValidator())

test_cases = ["", "ab", "user@", "user @example.com", "user@example.com"]

for email in test_cases:
    is_valid, error = email_validator.validate(email)
    print(f"'{email}': {'✓ Valid' if is_valid else '✗ ' + error}")
```

### Cas d'usage concret : Gestion des erreurs HTTP

```python
from abc import ABC, abstractmethod

class ErrorHandler(ABC):
    def __init__(self):
        self._next_handler = None
    
    def set_next(self, handler: 'ErrorHandler') -> 'ErrorHandler':
        self._next_handler = handler
        return handler
    
    @abstractmethod
    def handle(self, error_code: int, error_message: str) -> bool:
        if self._next_handler:
            return self._next_handler.handle(error_code, error_message)
        return False

class NotFoundHandler(ErrorHandler):
    def handle(self, error_code: int, error_message: str) -> bool:
        if error_code == 404:
            print(f"404 Handler: {error_message}")
            return True
        return super().handle(error_code, error_message)

class UnauthorizedHandler(ErrorHandler):
    def handle(self, error_code: int, error_message: str) -> bool:
        if error_code == 401:
            print(f"401 Handler: Redirecting to login - {error_message}")
            return True
        return super().handle(error_code, error_message)

class ServerErrorHandler(ErrorHandler):
    def handle(self, error_code: int, error_message: str) -> bool:
        if 500 <= error_code < 600:
            print(f"500 Handler: Logging error and showing generic message - {error_message}")
            return True
        return super().handle(error_code, error_message)

class DefaultHandler(ErrorHandler):
    def handle(self, error_code: int, error_message: str) -> bool:
        print(f"Default Handler: Unknown error {error_code}: {error_message}")
        return True

# Utilisation
handler_chain = NotFoundHandler()
handler_chain.set_next(UnauthorizedHandler()).set_next(ServerErrorHandler()).set_next(DefaultHandler())

errors = [
    (404, "Page not found"),
    (401, "Authentication required"),
    (500, "Internal server error"),
    (403, "Forbidden"),
]

for code, message in errors:
    handler_chain.handle(code, message)
```

### Quand utiliser Chain of Responsibility ?

✅ **Utilisez-le quand** :
- Plusieurs objets peuvent traiter une requête
- Vous voulez spécifier les handlers dynamiquement
- Vous voulez éviter de coupler l'émetteur aux récepteurs

---

## 5. State

### Intention

Permettre à un objet de modifier son comportement lorsque son état interne change. L'objet apparaîtra comme ayant changé de classe.

### Problème résolu

Comment gérer les différents comportements d'un objet selon son état sans utiliser de conditions multiples ?

### Solution

```python
from abc import ABC, abstractmethod

class State(ABC):
    @abstractmethod
    def handle(self, context: 'Context'):
        pass

class Context:
    def __init__(self):
        self._state = None
    
    def set_state(self, state: State):
        self._state = state
    
    def request(self):
        self._state.handle(self)

class ConcreteStateA(State):
    def handle(self, context: Context):
        print("State A: Handling request and transitioning to State B")
        context.set_state(ConcreteStateB())

class ConcreteStateB(State):
    def handle(self, context: Context):
        print("State B: Handling request and transitioning to State A")
        context.set_state(ConcreteStateA())

# Utilisation
context = Context()
context.set_state(ConcreteStateA())

context.request()  # State A -> State B
context.request()  # State B -> State A
context.request()  # State A -> State B
```

### Cas d'usage concret : Lecteur multimédia

```python
from abc import ABC, abstractmethod

class MediaPlayerState(ABC):
    @abstractmethod
    def play(self, player: 'MediaPlayer'):
        pass
    
    @abstractmethod
    def pause(self, player: 'MediaPlayer'):
        pass
    
    @abstractmethod
    def stop(self, player: 'MediaPlayer'):
        pass

class StoppedState(MediaPlayerState):
    def play(self, player: 'MediaPlayer'):
        print("Starting playback")
        player.set_state(PlayingState())
    
    def pause(self, player: 'MediaPlayer'):
        print("Cannot pause: player is stopped")
    
    def stop(self, player: 'MediaPlayer'):
        print("Player is already stopped")

class PlayingState(MediaPlayerState):
    def play(self, player: 'MediaPlayer'):
        print("Already playing")
    
    def pause(self, player: 'MediaPlayer'):
        print("Pausing playback")
        player.set_state(PausedState())
    
    def stop(self, player: 'MediaPlayer'):
        print("Stopping playback")
        player.set_state(StoppedState())

class PausedState(MediaPlayerState):
    def play(self, player: 'MediaPlayer'):
        print("Resuming playback")
        player.set_state(PlayingState())
    
    def pause(self, player: 'MediaPlayer'):
        print("Already paused")
    
    def stop(self, player: 'MediaPlayer'):
        print("Stopping playback")
        player.set_state(StoppedState())

class MediaPlayer:
    def __init__(self):
        self._state = StoppedState()
    
    def set_state(self, state: MediaPlayerState):
        self._state = state
    
    def play(self):
        self._state.play(self)
    
    def pause(self):
        self._state.pause(self)
    
    def stop(self):
        self._state.stop(self)

# Utilisation
player = MediaPlayer()

player.play()   # Starting playback
player.pause()  # Pausing playback
player.play()   # Resuming playback
player.stop()   # Stopping playback
player.pause()  # Cannot pause: player is stopped
```

### Cas d'usage concret : Machine à états d'une commande

```python
from abc import ABC, abstractmethod

class OrderState(ABC):
    @abstractmethod
    def cancel(self, order: 'Order'):
        pass
    
    @abstractmethod
    def ship(self, order: 'Order'):
        pass
    
    @abstractmethod
    def deliver(self, order: 'Order'):
        pass

class PendingState(OrderState):
    def cancel(self, order: 'Order'):
        print("Order cancelled")
        order.set_state(CancelledState())
    
    def ship(self, order: 'Order'):
        print("Order shipped")
        order.set_state(ShippedState())
    
    def deliver(self, order: 'Order'):
        print("Cannot deliver: order not shipped yet")

class ShippedState(OrderState):
    def cancel(self, order: 'Order'):
        print("Cannot cancel: order already shipped")
    
    def ship(self, order: 'Order'):
        print("Order already shipped")
    
    def deliver(self, order: 'Order'):
        print("Order delivered")
        order.set_state(DeliveredState())

class DeliveredState(OrderState):
    def cancel(self, order: 'Order'):
        print("Cannot cancel: order already delivered")
    
    def ship(self, order: 'Order'):
        print("Order already delivered")
    
    def deliver(self, order: 'Order'):
        print("Order already delivered")

class CancelledState(OrderState):
    def cancel(self, order: 'Order'):
        print("Order already cancelled")
    
    def ship(self, order: 'Order'):
        print("Cannot ship: order is cancelled")
    
    def deliver(self, order: 'Order'):
        print("Cannot deliver: order is cancelled")

class Order:
    def __init__(self):
        self._state = PendingState()
    
    def set_state(self, state: OrderState):
        self._state = state
    
    def cancel(self):
        self._state.cancel(self)
    
    def ship(self):
        self._state.ship(self)
    
    def deliver(self):
        self._state.deliver(self)

# Utilisation
order = Order()

order.ship()    # Order shipped
order.deliver() # Order delivered
order.cancel()  # Cannot cancel: order already delivered

order2 = Order()
order2.cancel() # Order cancelled
order2.ship()  # Cannot ship: order is cancelled
```

### Quand utiliser State ?

✅ **Utilisez-le quand** :
- Un objet a des comportements différents selon son état
- Vous avez beaucoup de conditions basées sur l'état
- Les transitions d'état sont bien définies

---

## Résumé des Patterns Comportementaux

| Pattern | Problème résolu | Quand l'utiliser |
|---------|----------------|------------------|
| **Observer** | Notifications d'événements | Systèmes d'événements, MVC |
| **Strategy** | Algorithmes interchangeables | Calculs, tri, validation |
| **Command** | Encapsuler des requêtes | Undo/redo, files d'attente |
| **Chain of Responsibility** | Traitement en chaîne | Validation, gestion d'erreurs |
| **State** | Comportement selon l'état | Machines à états, workflows |

---

**Prochaine étape** : Découvrez des [Exemples Concrets](./05-exemples-concrets.md) d'application de ces patterns dans des projets réels.
