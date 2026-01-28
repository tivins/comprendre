# Concepts fondamentaux du DDD

## 1. Le Domaine (Domain)

Le **domaine** est la sphère de connaissances ou d'activité autour de laquelle l'application est construite. C'est le cœur métier de votre application.

### Exemple concret

Pour une application de **gestion de bibliothèque** :
- Le domaine inclut : livres, emprunts, membres, réservations, retours, amendes
- Ce qui n'est PAS le domaine : base de données, interfaces utilisateur, APIs REST

## 2. Le Modèle du Domaine (Domain Model)

Le modèle du domaine est une abstraction qui représente les concepts et règles métier. C'est la représentation mentale partagée entre les développeurs et les experts métier.

### Caractéristiques importantes

- **Riche en comportement** : Le modèle contient la logique métier, pas seulement des données
- **Expressif** : Le code reflète le langage métier (Ubiquitous Language)
- **Isolé** : Le modèle est indépendant de l'infrastructure technique

## 3. Le Langage Ubiquitaire (Ubiquitous Language)

Le **langage ubiquitaire** est un vocabulaire commun utilisé par tous les membres de l'équipe (développeurs, experts métier, testeurs) pour parler du domaine.

### Pourquoi c'est important ?

- Évite les malentendus entre équipes
- Le code devient auto-documenté
- Facilite la communication

### Exemple

Dans un système de **e-commerce** :
- Bon : `Order`, `OrderItem`, `Payment`, `Shipment`
- À éviter : `OrderDTO`, `OrderEntity`, `OrderData`

## 4. Entités (Entities)

Une **entité** est un objet identifié par un identifiant unique, pas par ses attributs. Deux entités avec les mêmes valeurs mais des identifiants différents sont considérées comme différentes.

### Exemple

```python
class Book:
    def __init__(self, isbn: str, title: str, author: str):
        self.isbn = isbn  # Identifiant unique
        self.title = title
        self.author = author
    
    def __eq__(self, other):
        if not isinstance(other, Book):
            return False
        return self.isbn == other.isbn  # Comparaison par ISBN
```

Dans cet exemple, deux livres avec le même ISBN sont considérés comme le même livre, même si le titre ou l'auteur diffèrent (erreur de saisie).

## 5. Objets Valeur (Value Objects)

Un **objet valeur** est défini uniquement par ses attributs. Deux objets valeur avec les mêmes attributs sont considérés comme identiques et interchangeables.

### Caractéristiques

- **Immutables** : Une fois créé, un objet valeur ne peut pas être modifié
- **Pas d'identité** : Comparés par leurs valeurs
- **Réutilisables** : Peuvent être partagés sans risque

### Exemple

```python
class Money:
    def __init__(self, amount: float, currency: str):
        self.amount = amount
        self.currency = currency
    
    def __eq__(self, other):
        if not isinstance(other, Money):
            return False
        return self.amount == other.amount and self.currency == other.currency
    
    def add(self, other: 'Money') -> 'Money':
        if self.currency != other.currency:
            raise ValueError("Cannot add different currencies")
        return Money(self.amount + other.amount, self.currency)

# Utilisation
price1 = Money(100.0, "EUR")
price2 = Money(100.0, "EUR")
print(price1 == price2)  # True - même valeur
```

## 6. Agrégats (Aggregates)

Un **agrégat** est un cluster d'entités et d'objets valeur traités comme une unité. Il a une frontière claire et un **agrégat racine** (Aggregate Root) qui contrôle l'accès aux objets internes.

### Règles importantes

- Seul l'agrégat racine peut être référencé depuis l'extérieur
- Les objets à l'intérieur de l'agrégat ne peuvent être modifiés que via l'agrégat racine
- L'agrégat garantit l'intégrité et la cohérence des données

### Exemple

```python
class Order:  # Agrégat racine
    def __init__(self, order_id: str, customer_id: str):
        self.order_id = order_id
        self.customer_id = customer_id
        self._items = []  # Privé - accès contrôlé
        self.status = "Draft"
    
    def add_item(self, product_id: str, quantity: int, price: Money):
        if self.status != "Draft":
            raise ValueError("Cannot modify confirmed order")
        
        item = OrderItem(product_id, quantity, price)
        self._items.append(item)
    
    def confirm(self):
        if not self._items:
            raise ValueError("Cannot confirm empty order")
        self.status = "Confirmed"
    
    def get_total(self) -> Money:
        total = Money(0.0, "EUR")
        for item in self._items:
            total = total.add(item.get_subtotal())
        return total

class OrderItem:  # Entité interne à l'agrégat
    def __init__(self, product_id: str, quantity: int, price: Money):
        self.product_id = product_id
        self.quantity = quantity
        self.price = price
    
    def get_subtotal(self) -> Money:
        return Money(self.quantity * self.price.amount, self.price.currency)
```

## 7. Services du Domaine (Domain Services)

Un **service du domaine** contient une opération qui ne correspond naturellement à aucune entité ou objet valeur. C'est une opération qui implique plusieurs objets du domaine.

### Quand utiliser un service du domaine ?

- L'opération est une action importante du domaine
- L'opération ne correspond pas naturellement à une entité
- L'opération implique plusieurs agrégats

### Exemple

```python
class TransferService:  # Service du domaine
    def transfer(self, from_account: Account, to_account: Account, amount: Money):
        if from_account.balance.amount < amount.amount:
            raise ValueError("Insufficient funds")
        
        if from_account.currency != to_account.currency:
            # Logique de conversion de devise
            converted_amount = self._convert_currency(amount, to_account.currency)
            from_account.withdraw(amount)
            to_account.deposit(converted_amount)
        else:
            from_account.withdraw(amount)
            to_account.deposit(amount)
    
    def _convert_currency(self, money: Money, target_currency: str) -> Money:
        # Logique de conversion
        pass
```

## 8. Répositories (Repositories)

Un **répository** abstrait l'accès aux données. Il fournit une interface pour récupérer et stocker des agrégats, masquant les détails de persistance.

### Avantages

- Séparation entre domaine et infrastructure
- Facilite les tests (mocks)
- Interface simple et expressive

### Exemple

```python
class OrderRepository:
    def save(self, order: Order):
        """Sauvegarde une commande"""
        pass
    
    def find_by_id(self, order_id: str) -> Order:
        """Trouve une commande par son ID"""
        pass
    
    def find_by_customer(self, customer_id: str) -> list[Order]:
        """Trouve toutes les commandes d'un client"""
        pass
```

## Résumé

| Concept | Description | Exemple |
|---------|-------------|---------|
| **Domaine** | Sphère de connaissances métier | Gestion de bibliothèque |
| **Modèle du Domaine** | Abstraction représentant le domaine | Classes Book, Loan, Member |
| **Langage Ubiquitaire** | Vocabulaire commun | Termes métier utilisés partout |
| **Entité** | Objet avec identité unique | Book (identifié par ISBN) |
| **Objet Valeur** | Objet défini par ses attributs | Money, Address |
| **Agrégat** | Cluster d'objets avec frontière | Order avec OrderItems |
| **Service du Domaine** | Opération ne correspondant à aucune entité | TransferService |
| **Répository** | Abstraction de l'accès aux données | OrderRepository |

---

**Prochaine étape** : [Patterns tactiques](./02-patterns-tactiques.md)
