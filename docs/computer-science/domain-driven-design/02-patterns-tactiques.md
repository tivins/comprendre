# Patterns tactiques du DDD

Les **patterns tactiques** sont des patterns de conception utilisés au niveau du code pour structurer le modèle du domaine. Ils aident à créer un modèle expressif et maintenable.

## 1. Specification Pattern

Le pattern **Specification** encapsule une règle métier dans un objet réutilisable et testable.

### Problème

Souvent, les règles métier sont dispersées dans le code, difficiles à tester et à réutiliser.

### Solution

Créer des objets Specification qui encapsulent les règles métier.

### Exemple concret : Système de prêt de bibliothèque

```python
from abc import ABC, abstractmethod

class Specification(ABC):
    @abstractmethod
    def is_satisfied_by(self, candidate) -> bool:
        pass
    
    def and_specification(self, other: 'Specification') -> 'Specification':
        return AndSpecification(self, other)
    
    def or_specification(self, other: 'Specification') -> 'Specification':
        return OrSpecification(self, other)

class MemberCanBorrowSpecification(Specification):
    def __init__(self, max_loans: int = 5):
        self.max_loans = max_loans
    
    def is_satisfied_by(self, member: 'Member') -> bool:
        return (
            member.is_active() and
            member.get_active_loans_count() < self.max_loans and
            not member.has_overdue_books()
        )

class BookIsAvailableSpecification(Specification):
    def is_satisfied_by(self, book: 'Book') -> bool:
        return book.is_available() and not book.is_reserved()

# Combinaison de spécifications
class AndSpecification(Specification):
    def __init__(self, spec1: Specification, spec2: Specification):
        self.spec1 = spec1
        self.spec2 = spec2
    
    def is_satisfied_by(self, candidate) -> bool:
        return (
            self.spec1.is_satisfied_by(candidate) and
            self.spec2.is_satisfied_by(candidate)
        )

# Utilisation
member = Member(...)
book = Book(...)

can_borrow = MemberCanBorrowSpecification()
is_available = BookIsAvailableSpecification()

if can_borrow.and_specification(is_available).is_satisfied_by(member):
    # Le membre peut emprunter ce livre
    loan_service.create_loan(member, book)
```

### Avantages

- ✅ Règles métier réutilisables
- ✅ Facilement testables
- ✅ Peuvent être combinées
- ✅ Code expressif et lisible

## 2. Factory Pattern

Le pattern **Factory** encapsule la logique de création d'objets complexes, surtout pour les agrégats.

### Exemple concret : Création de commande e-commerce

```python
class OrderFactory:
    def create_order(self, customer_id: str, items_data: list[dict]) -> Order:
        """
        Crée une commande avec validation et règles métier
        """
        # Validation initiale
        if not items_data:
            raise ValueError("Order must have at least one item")
        
        # Création de l'agrégat racine
        order = Order(
            order_id=self._generate_order_id(),
            customer_id=customer_id,
            created_at=datetime.now()
        )
        
        # Ajout des items avec validation
        for item_data in items_data:
            product = self._product_repository.find_by_id(item_data['product_id'])
            
            if not product.is_available():
                raise ValueError(f"Product {product.id} is not available")
            
            price = product.get_price_for_quantity(item_data['quantity'])
            order.add_item(
                product_id=product.id,
                quantity=item_data['quantity'],
                price=price
            )
        
        # Application des règles métier
        order.apply_discount_if_eligible()
        
        return order
    
    def _generate_order_id(self) -> str:
        return f"ORD-{uuid.uuid4().hex[:8].upper()}"
```

### Avantages

- ✅ Encapsule la complexité de création
- ✅ Garantit la cohérence de l'agrégat
- ✅ Applique les règles métier à la création

## 3. Domain Events

Les **Domain Events** représentent quelque chose d'important qui s'est produit dans le domaine. Ils permettent de découpler les composants et de réagir aux changements.

### Exemple concret : Système de réservation

```python
from dataclasses import dataclass
from datetime import datetime
from typing import Any

@dataclass
class DomainEvent:
    occurred_on: datetime
    event_id: str

@dataclass
class OrderConfirmed(DomainEvent):
    order_id: str
    customer_id: str
    total_amount: float
    items_count: int

@dataclass
class PaymentReceived(DomainEvent):
    order_id: str
    amount: float
    payment_method: str

class Order:
    def __init__(self, order_id: str, customer_id: str):
        self.order_id = order_id
        self.customer_id = customer_id
        self.status = "Draft"
        self._domain_events = []  # Liste des événements
    
    def confirm(self):
        if self.status != "Draft":
            raise ValueError("Order already confirmed")
        
        self.status = "Confirmed"
        
        # Publication de l'événement
        event = OrderConfirmed(
            occurred_on=datetime.now(),
            event_id=str(uuid.uuid4()),
            order_id=self.order_id,
            customer_id=self.customer_id,
            total_amount=self.get_total().amount,
            items_count=len(self._items)
        )
        self._domain_events.append(event)
    
    def get_domain_events(self) -> list[DomainEvent]:
        return self._domain_events.copy()
    
    def clear_domain_events(self):
        self._domain_events.clear()

# Handler d'événements
class OrderConfirmedHandler:
    def handle(self, event: OrderConfirmed):
        # Envoyer un email de confirmation
        email_service.send_confirmation(event.customer_id, event.order_id)
        
        # Mettre à jour l'inventaire
        inventory_service.reserve_items(event.order_id)
        
        # Notifier le service de livraison
        shipping_service.prepare_shipment(event.order_id)
```

### Avantages

- ✅ Découplage entre agrégats
- ✅ Traçabilité des événements
- ✅ Facilite l'intégration avec d'autres systèmes
- ✅ Supporte l'Event Sourcing

## 4. Aggregate Root Pattern

L'**Aggregate Root** est l'entité qui contrôle l'accès à tous les objets de l'agrégat. C'est la seule interface publique de l'agrégat.

### Exemple concret : Panier d'achat

```python
class ShoppingCart:  # Aggregate Root
    def __init__(self, cart_id: str, customer_id: str):
        self.cart_id = cart_id
        self.customer_id = customer_id
        self._items = {}  # Dict privé
        self.created_at = datetime.now()
        self.updated_at = datetime.now()
    
    def add_item(self, product_id: str, quantity: int):
        """
        Seule méthode publique pour ajouter un item
        """
        if quantity <= 0:
            raise ValueError("Quantity must be positive")
        
        if product_id in self._items:
            # Mise à jour de la quantité existante
            self._items[product_id].update_quantity(
                self._items[product_id].quantity + quantity
            )
        else:
            # Création d'un nouvel item
            product = self._product_repository.find_by_id(product_id)
            if not product.is_available():
                raise ValueError("Product not available")
            
            self._items[product_id] = CartItem(
                product_id=product_id,
                quantity=quantity,
                unit_price=product.price
            )
        
        self.updated_at = datetime.now()
    
    def remove_item(self, product_id: str):
        if product_id not in self._items:
            raise ValueError("Item not in cart")
        
        del self._items[product_id]
        self.updated_at = datetime.now()
    
    def get_items(self) -> list['CartItem']:
        """
        Retourne une copie pour éviter la modification externe
        """
        return list(self._items.values())
    
    def get_total(self) -> Money:
        total = Money(0.0, "EUR")
        for item in self._items.values():
            total = total.add(item.get_subtotal())
        return total
    
    def clear(self):
        self._items.clear()
        self.updated_at = datetime.now()

class CartItem:  # Entité interne
    def __init__(self, product_id: str, quantity: int, unit_price: Money):
        self.product_id = product_id
        self._quantity = quantity
        self.unit_price = unit_price
    
    def update_quantity(self, new_quantity: int):
        """
        Méthode interne, appelée uniquement par l'agrégat racine
        """
        if new_quantity <= 0:
            raise ValueError("Quantity must be positive")
        self._quantity = new_quantity
    
    def get_subtotal(self) -> Money:
        return Money(self._quantity * self.unit_price.amount, self.unit_price.currency)
```

### Règles importantes

- ✅ Seul l'agrégat racine peut être référencé depuis l'extérieur
- ✅ Les objets internes ne sont accessibles que via l'agrégat racine
- ✅ L'agrégat garantit l'intégrité et la cohérence

## 5. Bounded Context Pattern

Un **Bounded Context** est une frontière explicite où un modèle du domaine particulier s'applique. À l'intérieur de cette frontière, tous les termes ont un sens précis.

### Exemple concret : Système de gestion d'entreprise

Dans une entreprise, le concept "Client" peut avoir des significations différentes selon le contexte :

**Contexte Ventes** :
```python
class Customer:  # Dans le contexte Ventes
    def __init__(self, customer_id: str, name: str, email: str):
        self.customer_id = customer_id
        self.name = name
        self.email = email
        self.total_purchases = 0.0
        self.loyalty_points = 0
    
    def make_purchase(self, amount: float):
        self.total_purchases += amount
        self.loyalty_points += int(amount / 10)
```

**Contexte Support** :
```python
class Customer:  # Dans le contexte Support
    def __init__(self, customer_id: str, name: str):
        self.customer_id = customer_id
        self.name = name
        self.open_tickets = []
        self.satisfaction_score = 0.0
    
    def open_ticket(self, issue: str):
        ticket = Ticket(issue=issue, customer_id=self.customer_id)
        self.open_tickets.append(ticket)
```

### Mapping entre contextes

```python
class CustomerMapper:
    """
    Mappe un Customer du contexte Ventes vers le contexte Support
    """
    @staticmethod
    def to_support_context(sales_customer: SalesCustomer) -> SupportCustomer:
        return SupportCustomer(
            customer_id=sales_customer.customer_id,
            name=sales_customer.name
        )
```

## Résumé des patterns tactiques

| Pattern | Objectif | Quand l'utiliser |
|---------|----------|-----------------|
| **Specification** | Encapsuler les règles métier | Règles complexes réutilisables |
| **Factory** | Créer des agrégats complexes | Création avec validation |
| **Domain Events** | Communiquer entre agrégats | Réactions aux changements |
| **Aggregate Root** | Contrôler l'accès à l'agrégat | Toujours pour les agrégats |
| **Bounded Context** | Délimiter les modèles | Systèmes complexes multi-domaines |

---

**Prochaine étape** : [Patterns stratégiques](./03-patterns-strategiques.md)
