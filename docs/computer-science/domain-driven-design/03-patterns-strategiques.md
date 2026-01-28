# Patterns stratégiques du DDD

Les **patterns stratégiques** concernent l'organisation et la structure à grande échelle de l'application. Ils aident à gérer la complexité en découpant le système en contextes délimités.

## 1. Bounded Context (Contexte Délimité)

Un **Bounded Context** est une frontière explicite où un modèle du domaine particulier s'applique. À l'intérieur, tous les termes ont un sens précis et partagé.

### Caractéristiques

- **Frontière claire** : Définit où le modèle s'applique
- **Modèle unique** : Un seul modèle du domaine par contexte
- **Langage ubiquitaire** : Vocabulaire spécifique au contexte
- **Indépendance** : Peut évoluer indépendamment

### Exemple concret : Plateforme e-commerce

```
┌─────────────────────┐     ┌─────────────────────┐     ┌─────────────────────┐
│   Catalogue         │     │   Commandes         │     │   Livraison         │
│   (Bounded Context) │     │   (Bounded Context) │     │   (Bounded Context) │
├─────────────────────┤     ├─────────────────────┤     ├─────────────────────┤
│ Product             │     │ Order               │     │ Shipment            │
│ - id                │     │ - orderId           │     │ - shipmentId        │
│ - name              │     │ - customerId        │     │ - orderId           │
│ - price             │     │ - items[]           │     │ - address           │
│ - stock             │     │ - total             │     │ - status            │
│ - description       │     │ - status           │     │ - trackingNumber    │
└─────────────────────┘     └─────────────────────┘     └─────────────────────┘
```

Dans chaque contexte, le concept "Product" peut avoir des attributs différents :
- **Catalogue** : Product avec description, images, catégories
- **Commandes** : Product avec seulement id, nom, prix
- **Livraison** : Product avec dimensions, poids

## 2. Context Mapping

Le **Context Mapping** est une technique pour visualiser et gérer les relations entre les différents Bounded Contexts.

### Types de relations

#### 1. Partnership (Partenariat)
Deux équipes travaillent ensemble sur des contextes interdépendants.

```
┌──────────────┐         ┌──────────────┐
│   Ventes     │◄───────►│   Marketing  │
│              │         │              │
└──────────────┘         └──────────────┘
```

#### 2. Shared Kernel (Noyau Partagé)
Deux équipes partagent un sous-ensemble du modèle.

```
┌──────────────┐         ┌──────────────┐
│   Ventes     │         │   Facturation│
│              │         │              │
└──────┬───────┘         └──────┬───────┘
       │                        │
       └──────────┬─────────────┘
                  │
         ┌────────▼────────┐
         │  Shared Kernel  │
         │  (Customer)     │
         └─────────────────┘
```

#### 3. Customer-Supplier (Client-Fournisseur)
Un contexte dépend d'un autre, mais le fournisseur ne dépend pas du client.

```
┌──────────────┐         ┌──────────────┐
│   Catalogue  │────────►│   Commandes  │
│  (Supplier)  │         │  (Customer)  │
└──────────────┘         └──────────────┘
```

#### 4. Conformist (Conformiste)
Un contexte doit se conformer au modèle d'un autre contexte (souvent externe).

```
┌──────────────┐         ┌──────────────┐
│   Système    │────────►│   Notre      │
│   Externe    │         │   Application│
│  (Legacy)    │         │ (Conformist) │
└──────────────┘         └──────────────┘
```

#### 5. Anti-Corruption Layer (Couche Anti-Corruption)
Une couche qui traduit entre deux modèles incompatibles.

```
┌──────────────┐         ┌──────────────┐         ┌──────────────┐
│   Système    │────────►│   ACL         │────────►│   Notre      │
│   Legacy     │         │  (Traduction)│         │   Domaine    │
└──────────────┘         └──────────────┘         └──────────────┘
```

### Exemple d'implémentation : Anti-Corruption Layer

```python
# Système legacy (modèle externe)
class LegacyCustomer:
    def __init__(self):
        self.cust_id = ""
        self.cust_name = ""
        self.cust_email = ""

# Notre modèle du domaine
class Customer:
    def __init__(self, customer_id: str, name: str, email: str):
        self.customer_id = customer_id
        self.name = name
        self.email = email

# Anti-Corruption Layer
class CustomerAdapter:
    """
    Adapte le modèle legacy vers notre modèle du domaine
    """
    @staticmethod
    def from_legacy(legacy_customer: LegacyCustomer) -> Customer:
        return Customer(
            customer_id=legacy_customer.cust_id,
            name=legacy_customer.cust_name,
            email=legacy_customer.cust_email
        )
    
    @staticmethod
    def to_legacy(customer: Customer) -> LegacyCustomer:
        legacy = LegacyCustomer()
        legacy.cust_id = customer.customer_id
        legacy.cust_name = customer.name
        legacy.cust_email = customer.email
        return legacy
```

## 3. Domain Services vs Application Services

### Domain Services
Contiennent la logique métier qui ne correspond à aucune entité spécifique.

```python
class PricingService:  # Domain Service
    """
    Calcule les prix selon les règles métier complexes
    """
    def calculate_price(
        self, 
        product: Product, 
        customer: Customer, 
        quantity: int
    ) -> Money:
        base_price = product.base_price
        
        # Règle métier : remise volume
        if quantity >= 100:
            base_price = base_price.multiply(0.9)  # 10% de réduction
        
        # Règle métier : remise fidélité
        if customer.is_vip():
            base_price = base_price.multiply(0.95)  # 5% supplémentaire
        
        return base_price.multiply(quantity)
```

### Application Services
Orchestrent les opérations entre plusieurs agrégats et services du domaine.

```python
class OrderApplicationService:  # Application Service
    def __init__(
        self,
        order_repository: OrderRepository,
        product_repository: ProductRepository,
        customer_repository: CustomerRepository,
        pricing_service: PricingService,
        payment_service: PaymentService
    ):
        self.order_repository = order_repository
        self.product_repository = product_repository
        self.customer_repository = customer_repository
        self.pricing_service = pricing_service
        self.payment_service = payment_service
    
    def create_order(self, customer_id: str, items: list[dict]) -> str:
        """
        Orchestre la création d'une commande
        """
        # 1. Récupérer les agrégats
        customer = self.customer_repository.find_by_id(customer_id)
        if not customer:
            raise ValueError("Customer not found")
        
        # 2. Créer la commande
        order = Order(order_id=str(uuid.uuid4()), customer_id=customer_id)
        
        # 3. Ajouter les items avec calcul de prix
        for item_data in items:
            product = self.product_repository.find_by_id(item_data['product_id'])
            
            # Utiliser le service du domaine pour calculer le prix
            price = self.pricing_service.calculate_price(
                product, 
                customer, 
                item_data['quantity']
            )
            
            order.add_item(
                product_id=product.id,
                quantity=item_data['quantity'],
                price=price
            )
        
        # 4. Sauvegarder
        self.order_repository.save(order)
        
        return order.order_id
    
    def process_payment(self, order_id: str, payment_data: dict):
        """
        Orchestre le paiement d'une commande
        """
        order = self.order_repository.find_by_id(order_id)
        if not order:
            raise ValueError("Order not found")
        
        # Traitement du paiement
        payment_result = self.payment_service.process(
            amount=order.get_total(),
            payment_data=payment_data
        )
        
        if payment_result.success:
            order.confirm()
            self.order_repository.save(order)
        
        return payment_result
```

### Différences clés

| Aspect | Domain Service | Application Service |
|--------|----------------|---------------------|
| **Responsabilité** | Logique métier pure | Orchestration |
| **Dépendances** | Autres objets du domaine | Repositories, Services |
| **Transaction** | Non | Oui (gère les transactions) |
| **Tests** | Tests unitaires | Tests d'intégration |

## 4. Event-Driven Architecture avec DDD

Les événements du domaine permettent de découpler les Bounded Contexts.

### Exemple : Système de gestion d'hôtel

```python
# Contexte Réservations
@dataclass
class ReservationCreated(DomainEvent):
    reservation_id: str
    room_id: str
    guest_id: str
    check_in: datetime
    check_out: datetime

class Reservation:
    def confirm(self):
        self.status = "Confirmed"
        event = ReservationCreated(
            occurred_on=datetime.now(),
            event_id=str(uuid.uuid4()),
            reservation_id=self.reservation_id,
            room_id=self.room_id,
            guest_id=self.guest_id,
            check_in=self.check_in,
            check_out=self.check_out
        )
        self._domain_events.append(event)

# Contexte Facturation (écoute l'événement)
class BillingEventHandler:
    def handle_reservation_created(self, event: ReservationCreated):
        # Créer une facture préliminaire
        invoice = Invoice.create_for_reservation(
            reservation_id=event.reservation_id,
            guest_id=event.guest_id,
            check_in=event.check_in,
            check_out=event.check_out
        )
        self.invoice_repository.save(invoice)

# Contexte Maintenance (écoute l'événement)
class MaintenanceEventHandler:
    def handle_reservation_created(self, event: ReservationCreated):
        # Planifier le nettoyage de la chambre
        cleaning_task = CleaningTask.create(
            room_id=event.room_id,
            scheduled_date=event.check_in
        )
        self.task_repository.save(cleaning_task)
```

## 5. CQRS (Command Query Responsibility Segregation)

CQRS sépare les opérations de lecture (Query) et d'écriture (Command).

### Architecture simple

```
┌──────────────┐         ┌──────────────┐
│   Commands   │         │    Queries   │
│   (Write)    │         │    (Read)    │
└──────┬───────┘         └──────┬───────┘
       │                        │
       ▼                        ▼
┌──────────────┐         ┌──────────────┐
│   Write DB   │         │   Read DB    │
│  (Normalized)│         │ (Denormalized)│
└──────────────┘         └──────────────┘
```

### Exemple

```python
# Command Side (Write)
class CreateOrderCommand:
    def __init__(self, customer_id: str, items: list[dict]):
        self.customer_id = customer_id
        self.items = items

class CreateOrderCommandHandler:
    def handle(self, command: CreateOrderCommand) -> str:
        order = Order.create(command.customer_id, command.items)
        self.order_repository.save(order)
        return order.order_id

# Query Side (Read)
class OrderQuery:
    def get_order_summary(self, order_id: str) -> dict:
        """
        Retourne une vue optimisée pour l'affichage
        """
        return self.read_db.query("""
            SELECT 
                o.id,
                o.customer_name,
                o.total,
                o.status,
                COUNT(oi.id) as items_count
            FROM order_summary_view o
            LEFT JOIN order_items oi ON o.id = oi.order_id
            WHERE o.id = ?
            GROUP BY o.id
        """, order_id)

class OrderSummaryView:
    """
    Vue dénormalisée optimisée pour les lectures
    """
    def __init__(self):
        self.order_id = ""
        self.customer_name = ""
        self.total = 0.0
        self.status = ""
        self.items_count = 0
```

## Résumé des patterns stratégiques

| Pattern | Objectif | Quand l'utiliser |
|---------|----------|-----------------|
| **Bounded Context** | Délimiter les modèles | Systèmes complexes |
| **Context Mapping** | Gérer les relations | Plusieurs contextes |
| **Anti-Corruption Layer** | Protéger le domaine | Intégration legacy |
| **Application Services** | Orchestrer les opérations | Cas d'usage complexes |
| **Event-Driven** | Découpler les contextes | Communication asynchrone |
| **CQRS** | Séparer lecture/écriture | Besoins de performance |

---

**Prochaine étape** : [Exemples concrets](./04-exemples-concrets.md)
