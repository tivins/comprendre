# Mise en pratique du DDD

Ce guide vous aidera à appliquer le DDD dans vos projets réels, étape par étape.

## 1. Identifier le domaine

### Questions à se poser

Avant de commencer, posez-vous ces questions :

1. **Quel est le cœur métier de l'application ?**
   - Quels sont les concepts principaux ?
   - Quelles sont les règles métier importantes ?

2. **Qui sont les experts du domaine ?**
   - Qui comprend le mieux le métier ?
   - Comment communiquer avec eux ?

3. **Quelle est la complexité ?**
   - Le domaine est-il simple ou complexe ?
   - Le DDD est-il justifié ?

### Exemple : Identification du domaine

**Application : Système de réservation de salles de réunion**

**Concepts identifiés :**
- Salle (Room)
- Réservation (Booking)
- Utilisateur (User)
- Horaire (TimeSlot)
- Équipement (Equipment)

**Règles métier identifiées :**
- Une salle ne peut être réservée qu'une seule fois pour un créneau horaire
- Les réservations doivent être confirmées dans les 24h
- Certaines salles nécessitent des autorisations spéciales
- Les réservations peuvent être récurrentes

## 2. Développer le langage ubiquitaire

### Processus

1. **Écouter** les experts métier
2. **Documenter** les termes utilisés
3. **Utiliser** ces termes dans le code
4. **Évoluer** le vocabulaire avec le projet

### Exemple : Glossaire

```markdown
# Glossaire du domaine - Système de réservation

## Termes principaux

- **Salle (Room)** : Espace physique pouvant être réservé
- **Réservation (Booking)** : Attribution d'une salle à un utilisateur pour un créneau
- **Créneau horaire (TimeSlot)** : Période de temps (début, fin)
- **Confirmation** : Validation d'une réservation provisoire
- **Récurrence** : Répétition d'une réservation (quotidienne, hebdomadaire, etc.)
- **Équipement** : Ressource disponible dans une salle (projecteur, tableau blanc, etc.)
```

### Application dans le code

```python
# Bon : Utilise le langage ubiquitaire
class Booking:
    def confirm(self):
        """Confirme la réservation"""
        pass

# À éviter : Termes techniques génériques
class ReservationEntity:
    def set_status_to_confirmed(self):
        """Change le statut"""
        pass
```

## 3. Modéliser le domaine

### Étapes de modélisation

1. **Identifier les entités et objets valeur**
2. **Définir les agrégats**
3. **Identifier les services du domaine**
4. **Définir les repositories**

### Template de modélisation

```python
# 1. Objets Valeur
@dataclass(frozen=True)
class TimeSlot:
    start: datetime
    end: datetime
    
    def overlaps(self, other: 'TimeSlot') -> bool:
        return self.start < other.end and self.end > other.start

# 2. Entités
class Room:
    def __init__(self, room_id: str, name: str, capacity: int):
        self.room_id = room_id
        self.name = name
        self.capacity = capacity

# 3. Agrégat
class Booking:
    def __init__(self, booking_id: str, room: Room, time_slot: TimeSlot):
        self.booking_id = booking_id
        self.room = room
        self.time_slot = time_slot
        self.status = "Pending"
    
    def confirm(self):
        if self.status != "Pending":
            raise ValueError("Only pending bookings can be confirmed")
        self.status = "Confirmed"

# 4. Service du Domaine
class BookingService:
    def is_room_available(self, room: Room, time_slot: TimeSlot) -> bool:
        # Logique de vérification de disponibilité
        pass

# 5. Repository
class BookingRepository:
    def save(self, booking: Booking):
        pass
    
    def find_by_room_and_time(self, room_id: str, time_slot: TimeSlot) -> Optional[Booking]:
        pass
```

## 4. Structurer le projet

### Organisation recommandée

```
project/
├── domain/                    # Couche domaine (cœur)
│   ├── entities/              # Entités
│   │   ├── room.py
│   │   └── booking.py
│   ├── value_objects/         # Objets valeur
│   │   ├── time_slot.py
│   │   └── address.py
│   ├── aggregates/           # Agrégats
│   │   └── booking_aggregate.py
│   ├── domain_services/      # Services du domaine
│   │   └── availability_service.py
│   ├── repositories/          # Interfaces de repositories
│   │   └── booking_repository.py
│   └── events/               # Événements du domaine
│       └── booking_confirmed.py
│
├── application/              # Couche application
│   ├── services/            # Services applicatifs
│   │   └── booking_application_service.py
│   └── use_cases/           # Cas d'usage
│       ├── create_booking.py
│       └── confirm_booking.py
│
├── infrastructure/          # Couche infrastructure
│   ├── persistence/         # Implémentation des repositories
│   │   └── sql_booking_repository.py
│   ├── messaging/           # Publication d'événements
│   └── external_services/   # Intégrations externes
│
└── presentation/            # Couche présentation
    ├── api/                 # API REST
    │   └── booking_controller.py
    └── cli/                 # Interface ligne de commande
        └── booking_commands.py
```

### Principes de dépendance

```
presentation → application → domain ← infrastructure
```

- La couche **domaine** ne dépend de rien
- Les autres couches dépendent du domaine
- L'infrastructure implémente les interfaces définies dans le domaine

## 5. Tests du domaine

### Stratégie de test

1. **Tests unitaires** pour le domaine
2. **Tests d'intégration** pour les services applicatifs
3. **Tests de bout en bout** pour les cas d'usage complets

### Exemple : Tests unitaires du domaine

```python
import pytest
from datetime import datetime, timedelta
from domain.entities.room import Room
from domain.value_objects.time_slot import TimeSlot
from domain.aggregates.booking import Booking

class TestBooking:
    def test_create_booking(self):
        """Test la création d'une réservation"""
        room = Room("R001", "Salle A", 10)
        time_slot = TimeSlot(
            start=datetime(2024, 1, 15, 10, 0),
            end=datetime(2024, 1, 15, 11, 0)
        )
        
        booking = Booking("B001", room, time_slot)
        
        assert booking.booking_id == "B001"
        assert booking.room == room
        assert booking.time_slot == time_slot
        assert booking.status == "Pending"
    
    def test_confirm_booking(self):
        """Test la confirmation d'une réservation"""
        room = Room("R001", "Salle A", 10)
        time_slot = TimeSlot(
            start=datetime(2024, 1, 15, 10, 0),
            end=datetime(2024, 1, 15, 11, 0)
        )
        booking = Booking("B001", room, time_slot)
        
        booking.confirm()
        
        assert booking.status == "Confirmed"
    
    def test_cannot_confirm_already_confirmed_booking(self):
        """Test qu'on ne peut pas confirmer une réservation déjà confirmée"""
        room = Room("R001", "Salle A", 10)
        time_slot = TimeSlot(
            start=datetime(2024, 1, 15, 10, 0),
            end=datetime(2024, 1, 15, 11, 0)
        )
        booking = Booking("B001", room, time_slot)
        booking.confirm()
        
        with pytest.raises(ValueError, match="Only pending bookings"):
            booking.confirm()
```

## 6. Éviter les pièges courants

### Piège 1 : Anémie du modèle

**Problème** : Modèle avec seulement des getters/setters, pas de comportement.

```python
# À éviter : Modèle anémique
class Order:
    def __init__(self):
        self.status = ""
        self.total = 0.0
    
    def set_status(self, status: str):
        self.status = status
    
    def set_total(self, total: float):
        self.total = total

# Bon : Modèle riche
class Order:
    def __init__(self, order_id: str):
        self.order_id = order_id
        self.status = "Draft"
        self._items = []
    
    def confirm(self):
        if not self._items:
            raise ValueError("Cannot confirm empty order")
        self.status = "Confirmed"
    
    def add_item(self, product_id: str, quantity: int, price: Money):
        if self.status != "Draft":
            raise ValueError("Cannot modify confirmed order")
        # Logique métier...
```

### Piège 2 : Fuite de logique métier

**Problème** : Logique métier dans les services applicatifs au lieu du domaine.

```python
# À éviter : Logique dans le service applicatif
class OrderService:
    def confirm_order(self, order_id: str):
        order = self.repository.find(order_id)
        # Logique métier ici (mauvais)
        if len(order.items) == 0:
            raise ValueError("Empty order")
        order.status = "Confirmed"
        self.repository.save(order)

# Bon : Logique dans le domaine
class OrderService:
    def confirm_order(self, order_id: str):
        order = self.repository.find(order_id)
        order.confirm()  # Logique métier dans le domaine
        self.repository.save(order)
```

### Piège 3 : Agrégats trop gros

**Problème** : Agrégat contenant trop d'entités, difficile à maintenir.

```python
# À éviter : Agrégat trop gros
class Order:
    def __init__(self):
        self.items = []
        self.payments = []
        self.shipments = []
        self.invoices = []
        self.notifications = []
        # ... trop de responsabilités

# Bon : Agrégats séparés
class Order:  # Agrégat principal
    def __init__(self):
        self.items = []
        # Seulement ce qui est nécessaire pour Order

class Payment:  # Agrégat séparé
    def __init__(self, order_id: str):
        self.order_id = order_id  # Référence, pas inclusion
```

## 7. Checklist de mise en pratique

### Phase 1 : Exploration
- [ ] Identifier les experts du domaine
- [ ] Créer un glossaire du langage ubiquitaire
- [ ] Documenter les règles métier principales
- [ ] Identifier les Bounded Contexts

### Phase 2 : Modélisation
- [ ] Identifier les entités et objets valeur
- [ ] Définir les agrégats et leurs racines
- [ ] Identifier les services du domaine
- [ ] Définir les interfaces des repositories

### Phase 3 : Implémentation
- [ ] Structurer le projet selon les couches
- [ ] Implémenter le modèle du domaine
- [ ] Implémenter les services applicatifs
- [ ] Implémenter les repositories

### Phase 4 : Tests
- [ ] Tests unitaires pour le domaine
- [ ] Tests d'intégration pour les services
- [ ] Tests de bout en bout

### Phase 5 : Refactoring
- [ ] Vérifier que le modèle reflète le domaine
- [ ] Simplifier les agrégats si nécessaire
- [ ] Améliorer le langage ubiquitaire

## 8. Ressources pour aller plus loin

### Livres
- **"Domain-Driven Design"** par Eric Evans (2003)
- **"Implementing Domain-Driven Design"** par Vaughn Vernon (2013)
- **"Domain-Driven Design Distilled"** par Vaughn Vernon (2016)

### Articles et blogs
- Site officiel de DDD : [domainlanguage.com](https://www.domainlanguage.com)
- Articles sur les patterns DDD
- Études de cas d'entreprises utilisant DDD

### Outils
- **EventStorming** : Technique de modélisation collaborative
- **Context Mapping** : Outils de visualisation des contextes
- **C4 Model** : Modélisation de l'architecture

## Conclusion

Le DDD n'est pas une solution universelle, mais une approche puissante pour gérer la complexité métier. Commencez petit, itérez, et laissez le modèle évoluer avec votre compréhension du domaine.

**Rappelez-vous** :
- Le domaine est au centre de tout
- Le langage ubiquitaire facilite la communication
- Les agrégats garantissent la cohérence
- Les tests valident votre modèle

---

**Retour à l'index** : [README](./README.md)
