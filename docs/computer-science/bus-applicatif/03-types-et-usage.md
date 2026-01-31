# Types et usage du bus applicatif

## 1. Command bus (bus de commandes)

### 1.1 Principe

Un **command bus** transporte des **commandes** : des messages qui expriment une **intention** (« créer une commande », « annuler un abonnement »). Chaque commande est traitée par **un seul** handler. Le handler modifie l’état du système (agrégats, base de données) et peut publier des **événements de domaine** pour signaler ce qui s’est passé.

### 1.2 Rôle dans le DDD

- Les commandes correspondent aux **cas d’usage** ou aux **actions** que l’utilisateur ou un autre bounded context peut déclencher.
- Le handler orchestre le **domaine** : il charge l’agrégat (ou en crée un), appelle des méthodes métier sur l’agrégat, persiste et publie les événements éventuels.
- L’écriture (CQRS) passe donc par le command bus : une intention → un handler → mise à jour du modèle d’écriture.

### 1.3 Exemple de flux

```
[Contrôleur / API]  →  dispatch(CreateOrder)  →  [Command Bus]
                                                        ↓
                                              [CreateOrderHandler]
                                                        ↓
                                              charge/crée agrégat Order
                                              Order::create(...)
                                              repository->save($order)
                                              eventBus->dispatch(OrderCreated)
```

## 2. Event bus (bus d’événements)

### 2.1 Principe

Un **event bus** transporte des **événements** : des messages qui décrivent un **fait accompli** (« une commande a été créée », « un utilisateur s’est inscrit »). Un même événement peut être traité par **zéro, un ou plusieurs** handlers. Les handlers ne modifient pas l’agrégat source ; ils réagissent (notifications, projections, intégrations).

### 2.2 Rôle dans le DDD

- Les événements sont des **événements de domaine** (Domain Events) : ils font partie du langage ubiquitaire et reflètent ce qui s’est passé dans un agrégat.
- Ils permettent de **découpler** les sous-domaines : le module « Commande » publie `OrderCreated` ; le module « Notification » ou « Statistiques » s’abonne sans que le module Commande ne les connaisse.
- En CQRS, les handlers d’événements mettent à jour les **projections** (modèles de lecture) ou envoient des données vers d’autres systèmes.

### 2.3 Exemple de flux

```
[CreateOrderHandler]  →  dispatch(OrderCreated)  →  [Event Bus]
                                                          ↓
                                    ┌─────────────────────┼─────────────────────┐
                                    ↓                     ↓                     ↓
                          [SendOrderConfirmation]  [UpdateOrderProjection]  [TrackAnalytics]
```

Plusieurs handlers réagissent au même événement, sans dépendance entre eux.

## 3. Un seul bus physique, deux usages logiques

En pratique, beaucoup de projets utilisent **un seul** bus (une seule abstraction « message bus ») avec deux types de messages :

- Les **commandes** : une seule destination (un handler), souvent exécutées en **synchrone** dans la requête HTTP pour que l’utilisateur reçoive le résultat (ID de la commande, erreur de validation).
- Les **événements** : plusieurs handlers possibles, souvent exécutés en **asynchrone** pour ne pas bloquer la réponse et pour permettre le scale (plusieurs workers).

La distinction se fait alors par le **type** du message (classe) et par la **configuration** du bus (routage, transports). Par exemple : toutes les classes implémentant `Command` vont au handler unique et en synchrone ; toutes les classes implémentant `DomainEvent` vont à la liste des handlers et peuvent être envoyées sur une queue.

## 4. CQRS : Command Query Responsibility Segregation

Le **CQRS** sépare clairement :

- **Command** (écriture) : modifier l’état ; passé par le **command bus** ; handler qui met à jour le modèle d’écriture (agrégats, tables d’écriture).
- **Query** (lecture) : ne pas modifier l’état ; peut aussi passer par un bus de requêtes (un handler par type de requête) qui interroge le **modèle de lecture** (projections, vues, caches).

Le bus applicatif sert donc surtout pour les **commandes** et les **événements**. Les requêtes peuvent rester des appels directs au modèle de lecture (repository de lecture, query handler) si vous ne formalisez pas un « query bus ». Beaucoup de projets ont un command bus + event bus et des query services ou query handlers appelés directement.

## 5. Intégration avec le DDD

### 5.1 Agrégats et commandes

- Une **commande** correspond à une action sur un ou plusieurs agrégats.
- Le **handler** charge l’agrégat (ou les agrégats) via le repository, appelle une méthode métier (ex. `Order::create`, `Subscription::cancel`), persiste et publie les événements de domaine éventuels.
- Les **événements de domaine** sont émis par l’agrégat (ou construits par le handler à partir de l’agrégat) et envoyés sur l’event bus.

### 5.2 Bounded contexts

- Chaque **bounded context** peut avoir son propre bus (ou son propre ensemble de messages et handlers).
- La communication entre contextes se fait par **événements** (ou messages partagés) : un contexte publie un événement, l’autre s’abonne. Le bus (ou un message broker) assure la livraison.

### 5.3 Langage ubiquitaire

- Les noms des **commandes** et des **événements** font partie du langage ubiquitaire : `CreateOrder`, `OrderCreated`, `CancelSubscription`, `SubscriptionCancelled`. Ils doivent être compris par les experts métier.

## Résumé

- **Command bus** : une commande = une intention → un handler → mise à jour du domaine (agrégats) et publication d’événements.
- **Event bus** : un événement = un fait → zéro à N handlers (notifications, projections, intégrations).
- Un même **bus physique** peut gérer commandes et événements ; la différence est dans le type de message et la configuration (routage, synchrone/asynchrone).
- En **CQRS**, le bus sert pour les commandes et les événements ; les requêtes peuvent être des query handlers ou des services de lecture.
- En **DDD**, les commandes pilotent les agrégats ; les événements de domaine circulent sur l’event bus pour découpler les bounded contexts et mettre à jour les projections.

Dans la suite, nous voyons des [exemples concrets](./04-exemples-concrets.md) en PHP avec Symfony Messenger et un modèle DDD.
