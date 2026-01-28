# Guide pratique : Questions à se poser lors de l'organisation du code

Ce guide vous aide à structurer votre code en vous posant les bonnes questions au bon moment. Il couvre les principes SOLID, la responsabilité unique, et les concepts DDD.

## Astuce mnémotechnique principale : **C.R.E.A.T.E**

**C** - Cohérence (une seule responsabilité)  
**R** - Réutilisabilité (découplage)  
**E** - Extensibilité (ouvert/fermé)  
**A** - Abstraction (interfaces)  
**T** - Testabilité (dépendances)  
**E** - Expressivité (nommage clair)

---

## Questions pour créer une CLASSE/OBJET

### 1. Responsabilité unique (SRP)

#### Questions essentielles :

1. **"Cette classe a-t-elle UNE SEULE raison de changer ?"**
   - Si vous répondez "oui" à plusieurs raisons → séparez en plusieurs classes
   - Exemple : `User` qui sauvegarde ET envoie des emails → 2 responsabilités

2. **"Puis-je décrire ce que fait cette classe en UNE phrase sans 'et' ?"**
   - "Cette classe représente un utilisateur"
   - "Cette classe représente un utilisateur ET le sauvegarde ET envoie des emails" (à éviter)

3. **"Si je dois modifier X, dois-je modifier cette classe ?"**
   - Si plusieurs X différents nécessitent des modifications → plusieurs responsabilités

4. **"Cette classe fait-elle partie du domaine métier ou de l'infrastructure ?"**
   - Domaine métier : `Order`, `Customer`, `Product`
   - Infrastructure : `DatabaseConnection`, `EmailService`, `FileStorage`

#### Exemples de code :

Code à éviter : Plusieurs responsabilités
```java
class User {
    private String name;
    private String email;
    
    public void save() {}
    public void sendEmail(String message) { }
    public void generateReport() { }
}
```

Meilleure version : Responsabilité unique
```java
class User { // Seulement la logique métier de l'utilisateur
    private String name;
    private String email;
}

class UserRepository {
    public void save(User user) { }
}

class EmailService {
    public void sendEmail(String to, String message) { }
}
```

#### Astuce mnémotechnique : **"UNE classe = UNE chose"**

Pensez à une classe comme un employé spécialisé :
- À éviter : Un employé qui fait la comptabilité ET la vente ET le marketing
- Préférable : Un comptable, un vendeur, un marketeur

---

### 2. Nommage et expressivité (DDD)

#### Questions essentielles :

1. **"Le nom de cette classe reflète-t-il un concept métier ?"**
   - Bon : `Order`, `Payment`, `ShippingAddress`
   - À éviter : `OrderManager`, `PaymentHandler`, `DataProcessor`

2. **"Un expert métier comprendrait-il ce nom ?"**
   - Si non, vous êtes peut-être trop technique

3. **"Cette classe est-elle un agrégat racine (DDD) ?"**
   - Si oui, elle contrôle l'accès à ses entités enfants
   - Exemple : `ShoppingCart` est l'agrégat racine, `CartItem` est une entité interne

#### Exemples de code :

```java
// À éviter : Noms trop techniques
class OrderManager {
    // ...
}

class PaymentHandler {
    // ...
}

class DataProcessor {
    // ...
}

// Bon : Noms métier clairs
class Order {
    private List<OrderItem> items;
    private OrderStatus status;
    
    public void addItem(Product product, int quantity) {
        // Logique métier de la commande
    }
}

class Payment {
    private Money amount;
    private PaymentMethod method;
    
    public void process() {
        // Logique métier du paiement
    }
}

// Exemple d'agrégat racine (DDD)
class ShoppingCart {  // Agrégat racine
    private List<CartItem> items;  // Entités internes
    
    public void addItem(Product product, int quantity) {
        // Contrôle l'accès aux items
        items.add(new CartItem(product, quantity));
    }
    
    public List<CartItem> getItems() {
        return Collections.unmodifiableList(items);
    }
}
```

#### Astuce mnémotechnique : **"Nom = Concept métier"**

Évitez les suffixes techniques (`Manager`, `Handler`, `Processor`) sauf si c'est vraiment nécessaire.

---

### 3. Dépendances et couplage (DIP)

#### Questions essentielles :

1. **"Cette classe dépend-elle d'une implémentation concrète ou d'une abstraction ?"**
   - Bon : Dépend d'une interface/abstraction
   - À éviter : Dépend directement d'une classe concrète

2. **"Puis-je tester cette classe sans instancier ses dépendances ?"**
   - Si non, utilisez l'injection de dépendances

3. **"Cette classe connaît-elle trop de détails sur ses dépendances ?"**
   - Elle ne devrait connaître que ce dont elle a besoin

#### Exemples de code :

```java
// À éviter : Dépendance du concret
class OrderService {
    private MySQLDatabase database;  // Dépend d'une implémentation spécifique
    
    public void saveOrder(Order order) {
        database.connect();
        database.insert("orders", order.toMap());
        database.close();
    }
}

// Bon : Dépendance de l'abstraction
interface OrderRepository {
    void save(Order order);
    Order findById(String id);
}

class OrderService {
    private OrderRepository repository;  // Dépend d'une interface
    
    public OrderService(OrderRepository repository) {
        this.repository = repository;
    }
    
    public void saveOrder(Order order) {
        repository.save(order);
    }
}

// Implémentations concrètes
class MySQLOrderRepository implements OrderRepository {
    public void save(Order order) {
        // Implémentation MySQL
    }
}

class MongoDBOrderRepository implements OrderRepository {
    public void save(Order order) {
        // Implémentation MongoDB
    }
}
```

#### Astuce mnémotechnique : **"Dépendre de l'abstraction, pas du concret"**

Pensez à une prise électrique :
- Bon : Elle accepte n'importe quel appareil (abstraction)
- À éviter : Elle n'accepte qu'un seul modèle d'appareil (concret)

---

## Questions pour créer une MÉTHODE

### 1. Responsabilité unique

#### Questions essentielles :

1. **"Cette méthode fait-elle UNE SEULE chose ?"**
   - Si elle fait plusieurs choses, décomposez-la

2. **"Puis-je nommer cette méthode avec un verbe simple ?"**
   - Bon : `calculateTotal()`, `validateOrder()`, `sendEmail()`
   - À éviter : `processAndValidateAndSaveOrder()` → 3 méthodes !

3. **"Cette méthode a-t-elle plus de 20 lignes ?"**
   - Si oui, elle fait probablement trop de choses

4. **"Y a-t-il des commentaires qui expliquent des sections de la méthode ?"**
   - Chaque section commentée devrait être une méthode séparée

#### Exemples de code :

```java
// À éviter : Méthode qui fait plusieurs choses
class OrderProcessor {
    public void processOrder(Order order) {
        // Valide la commande
        if (order.getItems().isEmpty()) {
            throw new IllegalArgumentException("Order is empty");
        }
        if (order.getTotal() <= 0) {
            throw new IllegalArgumentException("Invalid total");
        }
        
        // Calcule les taxes
        double tax = order.getTotal() * 0.20;
        order.setTax(tax);
        
        // Sauvegarde en base
        database.save(order);
        
        // Envoie un email
        emailService.send(order.getCustomer().getEmail(), "Order confirmed");
        
        // Génère une facture
        invoiceService.generate(order);
    }
}

// Bonne : Méthodes avec responsabilité unique
class OrderProcessor {
    public void processOrder(Order order) {
        validateOrder(order);
        calculateTax(order);
        saveOrder(order);
        notifyCustomer(order);
        generateInvoice(order);
    }
    
    private void validateOrder(Order order) {
        if (order.getItems().isEmpty()) {
            throw new IllegalArgumentException("Order is empty");
        }
        if (order.getTotal() <= 0) {
            throw new IllegalArgumentException("Invalid total");
        }
    }
    
    private void calculateTax(Order order) {
        double tax = order.getTotal() * 0.20;
        order.setTax(tax);
    }
    
    private void saveOrder(Order order) {
        repository.save(order);
    }
    
    private void notifyCustomer(Order order) {
        emailService.send(order.getCustomer().getEmail(), "Order confirmed");
    }
    
    private void generateInvoice(Order order) {
        invoiceService.generate(order);
    }
}
```

#### Astuce mnémotechnique : **"UNE méthode = UNE action"**

Pensez à une recette :
- À éviter : "Prépare ET cuit ET sert le plat"
- Préférable : `prepare()`, `cook()`, `serve()`

---

### 2. Paramètres et complexité

#### Questions essentielles :

1. **"Cette méthode a-t-elle plus de 3 paramètres ?"**
   - Si oui, créez un objet paramètre (Parameter Object Pattern)

2. **"Les paramètres sont-ils liés logiquement ?"**
   - Si oui, créez un objet valeur (Value Object)

3. **"Cette méthode retourne-t-elle plusieurs choses différentes ?"**
   - Si oui, créez un objet résultat

#### Exemples de code :

```java
// À éviter : Trop de paramètres
class UserService {
    public void createUser(String name, String email, String phone, 
                          String address, String city, String zipCode) {
        // Difficile à maintenir et à tester
    }
}

// Bon : Objet paramètre (Parameter Object)
class UserData {
    private String name;
    private String email;
    private String phone;
    private Address address;
    
    // Constructeur et getters
}

class Address {  // Value Object
    private String street;
    private String city;
    private String zipCode;
    
    // Constructeur et getters
}

class UserService {
    public void createUser(UserData userData) {
        // Plus clair et maintenable
    }
}

// Alternative : Builder Pattern pour plus de flexibilité
class UserDataBuilder {
    private String name;
    private String email;
    // ...
    
    public UserDataBuilder withName(String name) {
        this.name = name;
        return this;
    }
    
    public UserData build() {
        return new UserData(name, email, phone, address);
    }
}
```

#### Astuce mnémotechnique : **"3 paramètres max"**

Au-delà de 3 paramètres, créez un objet paramètre ou un Value Object.

---

### 3. Effets de bord et pureté

#### Questions essentielles :

1. **"Cette méthode modifie-t-elle l'état de l'objet ET retourne-t-elle une valeur ?"**
   - Préférez séparer : une méthode pour modifier, une pour lire

2. **"Cette méthode est-elle prévisible ?"**
   - Mêmes entrées → mêmes sorties (idéalement)

3. **"Cette méthode a-t-elle des effets de bord cachés ?"**
   - Exemple : modifier une variable globale, envoyer un email, etc.

#### Exemples de code :

```java
// À éviter : Mélange Command et Query
class ShoppingCart {
    private List<Item> items = new ArrayList<>();
    
    public List<Item> addItem(Item item) {
        items.add(item);
        return items;  // Retourne ET modifie !
    }
    
    public double calculateTotal() {
        double total = 0;
        for (Item item : items) {
            total += item.getPrice();
        }
        items.clear();  // Effet de bord caché !
        return total;
    }
}

// Bon : Séparation Command/Query
class ShoppingCart {
    private List<Item> items = new ArrayList<>();
    
    // Command : Modifie l'état, ne retourne rien (ou booléen)
    public void addItem(Item item) {
        items.add(item);
    }
    
    public boolean removeItem(Item item) {
        return items.remove(item);
    }
    
    // Query : Retourne une valeur, ne modifie rien
    public List<Item> getItems() {
        return Collections.unmodifiableList(items);
    }
    
    public double calculateTotal() {
        return items.stream()
            .mapToDouble(Item::getPrice)
            .sum();
    }
    
    public int getItemCount() {
        return items.size();
    }
}
```

#### Astuce mnémotechnique : **"Command/Query Separation"**

- **Command** : Modifie l'état (ne retourne rien ou un booléen)
- **Query** : Retourne une valeur (ne modifie rien)

---

## Questions pour créer un DOSSIER/MODULE

### 1. Organisation par responsabilité

#### Questions essentielles :

1. **"Ce dossier regroupe-t-il des classes avec la même responsabilité ?"**
   - Bon : `Domain/Order/` contient `Order`, `OrderItem`, `OrderStatus`
   - À éviter : `Utils/` contient tout et n'importe quoi

2. **"Ce dossier correspond-il à un contexte délimité (Bounded Context) ?"**
   - En DDD, chaque contexte a son propre modèle
   - Exemple : `Sales/`, `Shipping/`, `Support/`

3. **"Les classes de ce dossier changent-elles pour les mêmes raisons ?"**
   - Si oui, elles appartiennent au même dossier

#### Exemples de structure :

```java
// Bonne organisation : Par responsabilité métier
com.company.ecommerce.order/
    ├── Order.java              // Agrégat racine
    ├── OrderItem.java          // Entité
    ├── OrderStatus.java        // Value Object
    ├── OrderRepository.java    // Interface
    └── OrderService.java       // Service métier

com.company.ecommerce.payment/
    ├── Payment.java
    ├── PaymentMethod.java
    ├── PaymentRepository.java
    └── PaymentService.java

// À éviter : Mauvaise organisation : Tout mélangé
com.company.ecommerce.utils/
    ├── OrderHelper.java
    ├── PaymentHelper.java
    ├── EmailHelper.java
    ├── DatabaseHelper.java
    └── ValidationHelper.java
```

#### Astuce mnémotechnique : **"Même raison de changer = même dossier"**

Pensez à une bibliothèque :
- Bon : Tous les livres de cuisine sont ensemble
- À éviter : Tous les livres sont mélangés

---

### 2. Structure DDD

#### Questions essentielles :

1. **"Ce dossier représente-t-il une couche (Layer) ?"**
   - Domain, Application, Infrastructure, Presentation

2. **"Ce dossier représente-t-il un module métier ?"**
   - Exemple : `OrderManagement/`, `CustomerManagement/`

3. **"Les classes de ce dossier peuvent-elles être utilisées indépendamment ?"**
   - Si non, elles sont peut-être trop couplées

#### Exemples de structure DDD :

```java
// Structure par couches (Layered Architecture)
com.company.ecommerce/
    ├── domain/                    // D - Domain (métier pur)
    │   ├── order/
    │   │   ├── Order.java
    │   │   ├── OrderItem.java
    │   │   └── OrderRepository.java  // Interface
    │   └── customer/
    │       └── Customer.java
    │
    ├── application/               // A - Application (cas d'usage)
    │   ├── order/
    │   │   ├── CreateOrderUseCase.java
    │   │   └── CancelOrderUseCase.java
    │   └── customer/
    │       └── RegisterCustomerUseCase.java
    │
    ├── infrastructure/            // I - Infrastructure (technique)
    │   ├── persistence/
    │   │   ├── MySQLOrderRepository.java  // Implémente OrderRepository
    │   │   └── MongoDBOrderRepository.java
    │   ├── messaging/
    │   │   └── RabbitMQEventBus.java
    │   └── email/
    │       └── SMTPEmailService.java
    │
    └── presentation/              // P - Presentation (interface)
        ├── rest/
        │   └── OrderController.java
        └── graphql/
            └── OrderResolver.java
```

#### Astuce mnémotechnique : **"D.A.I.P"**

**D**omain (métier pur)  
**A**pplication (cas d'usage)  
**I**nfrastructure (technique)  
**P**resentation (interface)

Chaque couche ne dépend que des couches en dessous.

---

### 3. Couplage entre modules

#### Questions essentielles :

1. **"Ce dossier dépend-il directement d'un autre dossier ?"**
   - Domain ne doit pas dépendre d'Infrastructure

2. **"Y a-t-il des cycles de dépendances ?"**
   - Module A dépend de B, B dépend de A → problème !

3. **"Ce dossier expose-t-il trop de détails internes ?"**
   - Seules les interfaces publiques devraient être visibles

#### Exemples de dépendances :

```java
// Bon : Dépendances vers le bas
// Presentation dépend de Application
class OrderController {  // Presentation
    private CreateOrderUseCase useCase;  // Application
}

// Application dépend de Domain
class CreateOrderUseCase {  // Application
    private OrderRepository repository;  // Domain (interface)
}

// Infrastructure implémente Domain
class MySQLOrderRepository implements OrderRepository {  // Infrastructure
    // Implémente l'interface du Domain
}

// À éviter : Cycle de dépendances
class OrderService {  // Domain
    private EmailService emailService;  // Infrastructure
    // Domain ne doit PAS dépendre d'Infrastructure !
}

// Solution : Inversion de dépendance
interface EmailService {  // Dans Domain ou Application
    void sendEmail(String to, String message);
}

class SMTPEmailService implements EmailService {  // Infrastructure
    // Implémente l'interface
}
```

#### Astuce mnémotechnique : **"Dépendances vers le bas"**

Les dépendances doivent toujours aller vers les couches inférieures :
```
Presentation → Application → Domain
Infrastructure → Domain
```

---

## Checklist rapide avant de créer du code

### Pour une CLASSE

- [ ] **UNE** responsabilité claire
- [ ] Nom reflète un concept métier (si domaine)
- [ ] Dépend d'abstractions, pas de concret
- [ ] Peut être testée indépendamment
- [ ] Nom explicite (pas de `Manager`, `Handler` sauf nécessaire)
- [ ] Fait partie d'un contexte délimité clair

### Pour une MÉTHODE

- [ ] **UNE** action claire
- [ ] Nom verbe simple (`calculate`, `validate`, `send`)
- [ ] Moins de 3 paramètres (ou objet paramètre)
- [ ] Moins de 20 lignes (idéalement)
- [ ] Command OU Query, pas les deux
- [ ] Pas d'effets de bord cachés

### Pour un DOSSIER

- [ ] Regroupe des classes avec même responsabilité
- [ ] Correspond à un contexte délimité (DDD)
- [ ] Pas de cycles de dépendances
- [ ] Structure claire (Domain/Application/Infrastructure)
- [ ] Nom explicite du domaine métier

---

## Astuces mnémotechniques récapitulatives

### SOLID en une phrase chacun

**S** - **"UNE classe = UNE chose"**  
**O** - **"Ouvert à l'extension, fermé à la modification"**  
**L** - **"Les enfants doivent pouvoir remplacer les parents"**  
**I** - **"Interfaces petites et spécifiques"**  
**D** - **"Dépendre de l'abstraction, pas du concret"**

### DDD en une phrase chacun

**Aggregate Root** - **"Le chef d'orchestre de l'agrégat"**  
**Bounded Context** - **"Un modèle, un contexte, un sens"**  
**Value Object** - **"Égalité par valeur, pas par référence"**  
**Domain Event** - **"Quelque chose d'important s'est passé"**  
**Repository** - **"Collection d'objets en mémoire"**

### Règle des 3

- **3 responsabilités** → Séparer
- **3 paramètres** → Objet paramètre
- **3 niveaux d'imbrication** → Refactoriser
- **3 raisons de changer** → Séparer en 3 classes

---

## Questions de réflexion quotidiennes

### Avant d'écrire du code

1. **"Quel est le problème que je résous ?"**
   - Clarifie l'objectif

2. **"Qui va utiliser ce code ?"**
   - Clarifie l'interface publique

3. **"Qu'est-ce qui peut changer dans le futur ?"**
   - Identifie les abstractions nécessaires

### Pendant le développement

1. **"Est-ce que je peux tester cela facilement ?"**
   - Si non, trop de couplage

2. **"Est-ce que je comprends ce code dans 6 mois ?"**
   - Si non, trop complexe

3. **"Est-ce que je peux expliquer cela à un collègue en 30 secondes ?"**
   - Si non, trop abstrait ou mal nommé

### Après avoir écrit du code

1. **"Y a-t-il des duplications ?"**
   - DRY (Don't Repeat Yourself)

2. **"Y a-t-il des dépendances inutiles ?"**
   - Supprimez-les

3. **"Le nom reflète-t-il vraiment ce que fait le code ?"**
   - Si non, renommez

---

## Ressources pour aller plus loin

- [Principes SOLID](./../solid/README.md)
- [Domain-Driven Design](./../domain-driven-design/README.md)
- [Design Patterns](./../design-pattern/README.md)

---

**Rappel** : Ces principes sont des guides, pas des règles absolues. L'objectif est d'avoir un code maintenable et compréhensible. Parfois, une petite violation est acceptable si elle améliore la lisibilité ou la simplicité. L'important est de **savoir pourquoi** vous faites un choix.
