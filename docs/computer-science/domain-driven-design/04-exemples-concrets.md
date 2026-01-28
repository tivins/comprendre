# Exemples concrets de DDD

Cette section présente des exemples complets et concrets d'application du DDD dans différents domaines.

## Exemple 1 : Système de gestion de bibliothèque

### Contexte métier

Une bibliothèque municipale souhaite gérer :
- Les livres et leur disponibilité
- Les membres et leurs emprunts
- Les réservations
- Les amendes pour retards

### Modèle du domaine

```python
from dataclasses import dataclass
from datetime import datetime, timedelta
from typing import Optional
import uuid

# ========== Objets Valeur ==========

@dataclass(frozen=True)
class ISBN:
    """Objet valeur représentant un ISBN"""
    value: str
    
    def __post_init__(self):
        if not self._is_valid(self.value):
            raise ValueError(f"Invalid ISBN: {self.value}")
    
    @staticmethod
    def _is_valid(isbn: str) -> bool:
        # Validation simplifiée
        return len(isbn.replace('-', '')) in [10, 13]
    
    def __str__(self):
        return self.value

@dataclass(frozen=True)
class Address:
    """Objet valeur représentant une adresse"""
    street: str
    city: str
    postal_code: str
    country: str

# ========== Entités ==========

class Book:
    """Entité représentant un livre"""
    def __init__(self, isbn: ISBN, title: str, author: str, copies: int = 1):
        self.isbn = isbn  # Identifiant unique
        self.title = title
        self.author = author
        self._copies_total = copies
        self._copies_available = copies
        self._reservations = []
    
    def is_available(self) -> bool:
        """Vérifie si au moins un exemplaire est disponible"""
        return self._copies_available > 0
    
    def borrow_copy(self):
        """Emprunte un exemplaire"""
        if not self.is_available():
            raise ValueError("No copies available")
        self._copies_available -= 1
    
    def return_copy(self):
        """Retourne un exemplaire"""
        if self._copies_available >= self._copies_total:
            raise ValueError("Cannot return more copies than total")
        self._copies_available += 1
    
    def reserve(self, member_id: str):
        """Réserve le livre pour un membre"""
        if self.is_available():
            raise ValueError("Book is available, no need to reserve")
        if member_id in self._reservations:
            raise ValueError("Member already has a reservation")
        self._reservations.append(member_id)
    
    def cancel_reservation(self, member_id: str):
        """Annule une réservation"""
        if member_id not in self._reservations:
            raise ValueError("No reservation found")
        self._reservations.remove(member_id)
    
    def get_next_reservation(self) -> Optional[str]:
        """Retourne le prochain membre en liste d'attente"""
        return self._reservations[0] if self._reservations else None

class Member:
    """Entité représentant un membre de la bibliothèque"""
    def __init__(self, member_id: str, name: str, email: str, address: Address):
        self.member_id = member_id
        self.name = name
        self.email = email
        self.address = address
        self._active_loans = []
        self._is_active = True
        self._fines = Money(0.0, "EUR")
    
    def is_active(self) -> bool:
        return self._is_active
    
    def get_active_loans_count(self) -> int:
        return len(self._active_loans)
    
    def has_overdue_books(self) -> bool:
        return any(loan.is_overdue() for loan in self._active_loans)
    
    def add_loan(self, loan: 'Loan'):
        """Ajoute un emprunt"""
        self._active_loans.append(loan)
    
    def remove_loan(self, loan: 'Loan'):
        """Retire un emprunt"""
        if loan not in self._active_loans:
            raise ValueError("Loan not found")
        self._active_loans.remove(loan)
    
    def add_fine(self, amount: Money):
        """Ajoute une amende"""
        self._fines = self._fines.add(amount)
    
    def pay_fine(self, amount: Money):
        """Paie une partie de l'amende"""
        if amount.amount > self._fines.amount:
            raise ValueError("Cannot pay more than owed")
        self._fines = self._fines.subtract(amount)

# ========== Agrégat ==========

class Loan:
    """Agrégat racine représentant un emprunt"""
    def __init__(self, loan_id: str, member: Member, book: Book, loan_date: datetime):
        self.loan_id = loan_id
        self.member = member
        self.book = book
        self.loan_date = loan_date
        self.due_date = loan_date + timedelta(days=14)  # 2 semaines
        self.return_date: Optional[datetime] = None
        self.status = "Active"
    
    def is_overdue(self) -> bool:
        """Vérifie si l'emprunt est en retard"""
        if self.status == "Returned":
            return False
        return datetime.now() > self.due_date
    
    def calculate_fine(self) -> Money:
        """Calcule l'amende si l'emprunt est en retard"""
        if not self.is_overdue():
            return Money(0.0, "EUR")
        
        days_overdue = (datetime.now() - self.due_date).days
        fine_per_day = Money(0.50, "EUR")
        return fine_per_day.multiply(days_overdue)
    
    def return_book(self, return_date: datetime):
        """Retourne le livre"""
        if self.status == "Returned":
            raise ValueError("Loan already returned")
        
        self.return_date = return_date
        self.status = "Returned"
        self.book.return_copy()
        self.member.remove_loan(self)
        
        # Calculer et ajouter l'amende si nécessaire
        fine = self.calculate_fine()
        if fine.amount > 0:
            self.member.add_fine(fine)

# ========== Service du Domaine ==========

class LoanService:
    """Service du domaine pour gérer les emprunts"""
    MAX_LOANS_PER_MEMBER = 5
    
    def __init__(self, loan_repository, book_repository, member_repository):
        self.loan_repository = loan_repository
        self.book_repository = book_repository
        self.member_repository = member_repository
    
    def borrow_book(self, member_id: str, isbn: ISBN) -> str:
        """Emprunte un livre"""
        # Récupérer les agrégats
        member = self.member_repository.find_by_id(member_id)
        if not member:
            raise ValueError("Member not found")
        
        book = self.book_repository.find_by_isbn(isbn)
        if not book:
            raise ValueError("Book not found")
        
        # Vérifier les règles métier
        if not member.is_active():
            raise ValueError("Member is not active")
        
        if member.get_active_loans_count() >= self.MAX_LOANS_PER_MEMBER:
            raise ValueError(f"Member has reached maximum loans ({self.MAX_LOANS_PER_MEMBER})")
        
        if member.has_overdue_books():
            raise ValueError("Member has overdue books")
        
        if not book.is_available():
            # Réserver au lieu d'emprunter
            book.reserve(member_id)
            self.book_repository.save(book)
            raise ValueError("Book not available, reservation created")
        
        # Créer l'emprunt
        loan = Loan(
            loan_id=str(uuid.uuid4()),
            member=member,
            book=book,
            loan_date=datetime.now()
        )
        
        # Mettre à jour les agrégats
        book.borrow_copy()
        member.add_loan(loan)
        
        # Sauvegarder
        self.loan_repository.save(loan)
        self.book_repository.save(book)
        self.member_repository.save(member)
        
        return loan.loan_id
    
    def return_book(self, loan_id: str) -> Optional[Money]:
        """Retourne un livre"""
        loan = self.loan_repository.find_by_id(loan_id)
        if not loan:
            raise ValueError("Loan not found")
        
        fine = loan.calculate_fine()
        loan.return_book(datetime.now())
        
        # Si le livre était réservé, notifier le prochain membre
        if loan.book.get_next_reservation():
            # Envoyer une notification (via événement de domaine)
            pass
        
        self.loan_repository.save(loan)
        self.book_repository.save(loan.book)
        self.member_repository.save(loan.member)
        
        return fine if fine.amount > 0 else None

# ========== Repository (Interface) ==========

class LoanRepository:
    def save(self, loan: Loan):
        pass
    
    def find_by_id(self, loan_id: str) -> Optional[Loan]:
        pass
    
    def find_by_member(self, member_id: str) -> list[Loan]:
        pass
```

## Exemple 2 : Système de gestion de commandes e-commerce

### Contexte métier

Un site e-commerce doit gérer :
- Les produits et leur inventaire
- Les commandes avec plusieurs articles
- Les paiements
- Les expéditions

### Modèle du domaine

```python
from dataclasses import dataclass
from datetime import datetime
from enum import Enum
from typing import List
import uuid

# ========== Objets Valeur ==========

@dataclass(frozen=True)
class Money:
    amount: float
    currency: str
    
    def add(self, other: 'Money') -> 'Money':
        if self.currency != other.currency:
            raise ValueError("Cannot add different currencies")
        return Money(self.amount + other.amount, self.currency)
    
    def subtract(self, other: 'Money') -> 'Money':
        if self.currency != other.currency:
            raise ValueError("Cannot subtract different currencies")
        return Money(self.amount - other.amount, self.currency)
    
    def multiply(self, factor: float) -> 'Money':
        return Money(self.amount * factor, self.currency)

@dataclass(frozen=True)
class Address:
    street: str
    city: str
    postal_code: str
    country: str

# ========== Événements du Domaine ==========

@dataclass
class DomainEvent:
    occurred_on: datetime
    event_id: str

@dataclass
class OrderCreated(DomainEvent):
    order_id: str
    customer_id: str

@dataclass
class OrderConfirmed(DomainEvent):
    order_id: str
    customer_id: str
    total_amount: float

@dataclass
class OrderCancelled(DomainEvent):
    order_id: str
    reason: str

# ========== Agrégat ==========

class OrderStatus(Enum):
    DRAFT = "Draft"
    CONFIRMED = "Confirmed"
    PAID = "Paid"
    SHIPPED = "Shipped"
    DELIVERED = "Delivered"
    CANCELLED = "Cancelled"

class Order:
    """Agrégat racine représentant une commande"""
    def __init__(self, order_id: str, customer_id: str, created_at: datetime):
        self.order_id = order_id
        self.customer_id = customer_id
        self.created_at = created_at
        self._items: List['OrderItem'] = []
        self.status = OrderStatus.DRAFT
        self.shipping_address: Optional[Address] = None
        self._domain_events: List[DomainEvent] = []
    
    def add_item(self, product_id: str, quantity: int, unit_price: Money):
        """Ajoute un article à la commande"""
        if self.status != OrderStatus.DRAFT:
            raise ValueError("Cannot modify order in status: " + self.status.value)
        
        if quantity <= 0:
            raise ValueError("Quantity must be positive")
        
        # Vérifier si le produit existe déjà
        existing_item = next(
            (item for item in self._items if item.product_id == product_id),
            None
        )
        
        if existing_item:
            existing_item.update_quantity(existing_item.quantity + quantity)
        else:
            item = OrderItem(product_id, quantity, unit_price)
            self._items.append(item)
    
    def remove_item(self, product_id: str):
        """Retire un article de la commande"""
        if self.status != OrderStatus.DRAFT:
            raise ValueError("Cannot modify order in status: " + self.status.value)
        
        self._items = [item for item in self._items if item.product_id != product_id]
    
    def set_shipping_address(self, address: Address):
        """Définit l'adresse de livraison"""
        if self.status != OrderStatus.DRAFT:
            raise ValueError("Cannot modify order in status: " + self.status.value)
        self.shipping_address = address
    
    def get_total(self) -> Money:
        """Calcule le total de la commande"""
        total = Money(0.0, "EUR")
        for item in self._items:
            total = total.add(item.get_subtotal())
        return total
    
    def confirm(self):
        """Confirme la commande"""
        if self.status != OrderStatus.DRAFT:
            raise ValueError("Order already confirmed")
        
        if not self._items:
            raise ValueError("Cannot confirm empty order")
        
        if not self.shipping_address:
            raise ValueError("Shipping address required")
        
        self.status = OrderStatus.CONFIRMED
        
        event = OrderConfirmed(
            occurred_on=datetime.now(),
            event_id=str(uuid.uuid4()),
            order_id=self.order_id,
            customer_id=self.customer_id,
            total_amount=self.get_total().amount
        )
        self._domain_events.append(event)
    
    def mark_as_paid(self):
        """Marque la commande comme payée"""
        if self.status != OrderStatus.CONFIRMED:
            raise ValueError("Order must be confirmed before payment")
        self.status = OrderStatus.PAID
    
    def cancel(self, reason: str):
        """Annule la commande"""
        if self.status in [OrderStatus.SHIPPED, OrderStatus.DELIVERED]:
            raise ValueError("Cannot cancel shipped or delivered order")
        
        self.status = OrderStatus.CANCELLED
        
        event = OrderCancelled(
            occurred_on=datetime.now(),
            event_id=str(uuid.uuid4()),
            order_id=self.order_id,
            reason=reason
        )
        self._domain_events.append(event)
    
    def get_items(self) -> List['OrderItem']:
        """Retourne une copie des articles"""
        return list(self._items)
    
    def get_domain_events(self) -> List[DomainEvent]:
        """Retourne les événements du domaine"""
        return self._domain_events.copy()
    
    def clear_domain_events(self):
        """Efface les événements du domaine"""
        self._domain_events.clear()

class OrderItem:
    """Entité interne représentant un article de commande"""
    def __init__(self, product_id: str, quantity: int, unit_price: Money):
        self.product_id = product_id
        self._quantity = quantity
        self.unit_price = unit_price
    
    @property
    def quantity(self) -> int:
        return self._quantity
    
    def update_quantity(self, new_quantity: int):
        """Met à jour la quantité"""
        if new_quantity <= 0:
            raise ValueError("Quantity must be positive")
        self._quantity = new_quantity
    
    def get_subtotal(self) -> Money:
        """Calcule le sous-total"""
        return self.unit_price.multiply(self._quantity)

# ========== Service du Domaine ==========

class PricingService:
    """Service du domaine pour calculer les prix"""
    def calculate_discount(self, order: Order, customer_tier: str) -> Money:
        """Calcule la remise selon le niveau du client"""
        base_total = order.get_total()
        
        discounts = {
            "Bronze": 0.0,
            "Silver": 0.05,  # 5%
            "Gold": 0.10,   # 10%
            "Platinum": 0.15  # 15%
        }
        
        discount_rate = discounts.get(customer_tier, 0.0)
        return base_total.multiply(discount_rate)
    
    def apply_volume_discount(self, order: Order) -> Money:
        """Applique une remise volume"""
        total_items = sum(item.quantity for item in order.get_items())
        
        if total_items >= 10:
            return order.get_total().multiply(0.1)  # 10% de réduction
        elif total_items >= 5:
            return order.get_total().multiply(0.05)  # 5% de réduction
        
        return Money(0.0, "EUR")

# ========== Application Service ==========

class OrderApplicationService:
    """Service applicatif orchestrant les opérations"""
    def __init__(
        self,
        order_repository,
        product_repository,
        customer_repository,
        pricing_service: PricingService,
        inventory_service
    ):
        self.order_repository = order_repository
        self.product_repository = product_repository
        self.customer_repository = customer_repository
        self.pricing_service = pricing_service
        self.inventory_service = inventory_service
    
    def create_order(self, customer_id: str, items_data: List[dict]) -> str:
        """Crée une nouvelle commande"""
        # Vérifier que le client existe
        customer = self.customer_repository.find_by_id(customer_id)
        if not customer:
            raise ValueError("Customer not found")
        
        # Créer la commande
        order = Order(
            order_id=str(uuid.uuid4()),
            customer_id=customer_id,
            created_at=datetime.now()
        )
        
        # Ajouter les articles avec vérification de disponibilité
        for item_data in items_data:
            product = self.product_repository.find_by_id(item_data['product_id'])
            if not product:
                raise ValueError(f"Product {item_data['product_id']} not found")
            
            # Vérifier l'inventaire
            if not self.inventory_service.is_available(
                product.id, 
                item_data['quantity']
            ):
                raise ValueError(f"Insufficient inventory for product {product.id}")
            
            order.add_item(
                product_id=product.id,
                quantity=item_data['quantity'],
                unit_price=product.price
            )
        
        # Sauvegarder
        self.order_repository.save(order)
        
        return order.order_id
    
    def confirm_order(self, order_id: str, shipping_address: Address):
        """Confirme une commande"""
        order = self.order_repository.find_by_id(order_id)
        if not order:
            raise ValueError("Order not found")
        
        # Définir l'adresse
        order.set_shipping_address(shipping_address)
        
        # Vérifier et réserver l'inventaire
        for item in order.get_items():
            if not self.inventory_service.reserve(
                item.product_id,
                item.quantity
            ):
                raise ValueError(f"Cannot reserve inventory for product {item.product_id}")
        
        # Confirmer la commande
        order.confirm()
        
        # Sauvegarder
        self.order_repository.save(order)
        
        # Publier les événements
        events = order.get_domain_events()
        for event in events:
            self._publish_event(event)
        
        order.clear_domain_events()
    
    def _publish_event(self, event: DomainEvent):
        """Publie un événement du domaine"""
        # Implémentation de la publication d'événements
        pass
```

## Exemple 3 : Système de gestion de tâches collaboratif

### Contexte métier

Une application de gestion de projet doit gérer :
- Les projets et leurs membres
- Les tâches avec assignation
- Les commentaires et discussions
- Les deadlines et priorités

### Modèle simplifié

```python
from dataclasses import dataclass
from datetime import datetime
from enum import Enum
from typing import List, Set
import uuid

class Priority(Enum):
    LOW = "Low"
    MEDIUM = "Medium"
    HIGH = "High"
    URGENT = "Urgent"

class TaskStatus(Enum):
    TODO = "Todo"
    IN_PROGRESS = "In Progress"
    IN_REVIEW = "In Review"
    DONE = "Done"
    BLOCKED = "Blocked"

class Task:
    """Agrégat racine représentant une tâche"""
    def __init__(
        self,
        task_id: str,
        project_id: str,
        title: str,
        description: str,
        created_by: str
    ):
        self.task_id = task_id
        self.project_id = project_id
        self.title = title
        self.description = description
        self.created_by = created_by
        self.created_at = datetime.now()
        
        self.status = TaskStatus.TODO
        self.priority = Priority.MEDIUM
        self.assigned_to: Optional[str] = None
        self.due_date: Optional[datetime] = None
        
        self._comments: List['Comment'] = []
        self._tags: Set[str] = set()
    
    def assign(self, user_id: str):
        """Assigne la tâche à un utilisateur"""
        if self.status == TaskStatus.DONE:
            raise ValueError("Cannot assign completed task")
        self.assigned_to = user_id
        if self.status == TaskStatus.TODO:
            self.status = TaskStatus.IN_PROGRESS
    
    def add_comment(self, author_id: str, content: str):
        """Ajoute un commentaire"""
        comment = Comment(
            comment_id=str(uuid.uuid4()),
            task_id=self.task_id,
            author_id=author_id,
            content=content,
            created_at=datetime.now()
        )
        self._comments.append(comment)
    
    def change_status(self, new_status: TaskStatus):
        """Change le statut de la tâche"""
        # Règles de transition
        valid_transitions = {
            TaskStatus.TODO: [TaskStatus.IN_PROGRESS, TaskStatus.BLOCKED],
            TaskStatus.IN_PROGRESS: [TaskStatus.IN_REVIEW, TaskStatus.BLOCKED],
            TaskStatus.IN_REVIEW: [TaskStatus.DONE, TaskStatus.IN_PROGRESS],
            TaskStatus.BLOCKED: [TaskStatus.TODO, TaskStatus.IN_PROGRESS],
            TaskStatus.DONE: []  # État final
        }
        
        if new_status not in valid_transitions.get(self.status, []):
            raise ValueError(
                f"Cannot transition from {self.status.value} to {new_status.value}"
            )
        
        self.status = new_status
    
    def add_tag(self, tag: str):
        """Ajoute un tag"""
        self._tags.add(tag)
    
    def is_overdue(self) -> bool:
        """Vérifie si la tâche est en retard"""
        if not self.due_date:
            return False
        if self.status == TaskStatus.DONE:
            return False
        return datetime.now() > self.due_date
    
    def get_comments(self) -> List['Comment']:
        return list(self._comments)

class Comment:
    """Entité représentant un commentaire"""
    def __init__(
        self,
        comment_id: str,
        task_id: str,
        author_id: str,
        content: str,
        created_at: datetime
    ):
        self.comment_id = comment_id
        self.task_id = task_id
        self.author_id = author_id
        self.content = content
        self.created_at = created_at
        self.edited_at: Optional[datetime] = None
    
    def edit(self, new_content: str):
        """Modifie le commentaire"""
        self.content = new_content
        self.edited_at = datetime.now()
```

## Points clés à retenir

1. **Séparation claire** : Le modèle du domaine est indépendant de l'infrastructure
2. **Richesse comportementale** : Les objets du domaine contiennent la logique métier
3. **Invariants** : Les agrégats garantissent la cohérence des données
4. **Langage expressif** : Le code reflète le langage métier
5. **Événements** : Les événements du domaine permettent la communication découplée

---

**Prochaine étape** : [Mise en pratique](./05-mise-en-pratique.md)
