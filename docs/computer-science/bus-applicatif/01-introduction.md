# Introduction au bus applicatif

## Qu'est-ce qu'un bus applicatif ?

Un **bus applicatif** (ou **message bus**) est un canal logiciel partagé permettant d’envoyer des **messages** (commandes, événements, requêtes) entre les composants d’une application sans que l’émetteur ne connaisse directement le ou les destinataires. Il joue le rôle de « colonne vertébrale » des échanges côté application : au lieu d’appeler chaque service ou handler manuellement, on envoie un message sur le bus, qui le route vers le bon traitement.

### Analogie avec le bus matériel

On peut comparer un bus applicatif au **bus matériel** (CPU, mémoire, périphériques) :

- **Canal commun** : un même mécanisme (le bus) est utilisé par plusieurs composants (contrôleurs, services, handlers).
- **Messages au lieu de signaux** : les « données » sont des objets métier (commande, événement) ; la « destination » est déterminée par le type du message.
- **Découplage** : l’émetteur n’a pas besoin de savoir qui traite le message ; le bus assure le routage.
- **Règles de circulation** : un bus applicatif peut être synchrone (traitement immédiat) ou asynchrone (file d’attente, workers).

Sans bus applicatif, chaque contrôleur ou service appellerait directement les autres : le couplage serait fort et l’évolution difficile. Le bus **mutualise** la logique de dispatch et **découple** les composants.

## Rôle du bus dans l'architecture applicative

### 1. Découpler l’émetteur du récepteur

Sans bus, pour déclencher une action après une commande utilisateur, on ferait par exemple :

```php
// Couplage fort : le contrôleur connaît tous les services
$orderService->createOrder($data);
$inventoryService->reserveStock($data['items']);
$notificationService->sendConfirmation($user);
$analyticsService->trackOrder($data);
```

Avec un bus, on envoie une seule commande ; les handlers (éventuellement plusieurs pour un événement) sont invoqués par le bus :

```php
// Découplé : le contrôleur envoie une commande, le bus route
$bus->dispatch(new CreateOrder($data));
```

L’ajout d’un nouveau comportement (ex. envoi d’un SMS) se fait en enregistrant un nouveau handler, sans modifier le contrôleur.

### 2. Centraliser le flux (transactions, logging, sécurité)

Le bus permet d’envelopper chaque traitement dans des **middlewares** :

- **Transaction** : ouvrir une transaction avant le handler, committer ou rollback après.
- **Logging** : tracer chaque message entrant et sortant.
- **Sécurité** : vérifier les droits avant d’exécuter le handler.
- **Retry** : en cas d’échec, réessayer selon une stratégie définie.

Sans bus, cette logique serait dupliquée dans chaque service.

### 3. Séparer les préoccupations (CQRS, DDD)

On distingue souvent :

- **Command bus** : une commande = une intention (ex. « Créer une commande ») → un seul handler qui modifie l’état.
- **Event bus** : un événement = un fait accompli (ex. « Commande créée ») → zéro, un ou plusieurs handlers (notifications, projections, intégrations).

Cette séparation s’intègre naturellement au **Domain-Driven Design** : les commandes portent les intentions métier, les événements reflètent les changements d’état des agrégats.

## Message : la « monnaie » du bus

Sur un bus applicatif, tout ce qui circule est un **message**. On distingue en pratique :

| Type       | Rôle                    | Nombre de handlers | Exemple (PHP)           |
|-----------|-------------------------|--------------------|--------------------------|
| **Commande** | Intention (faire quelque chose) | Un seul             | `CreateOrder`, `CancelSubscription` |
| **Événement** | Fait accompli (quelque chose s’est passé) | Zéro à plusieurs    | `OrderCreated`, `UserRegistered`     |
| **Requête** (optionnel) | Lire des données (CQRS)  | Un seul             | `GetOrderById`, `ListProducts`       |

En PHP, un message est le plus souvent un **objet** (classe) portant des données ; le bus utilise le type (ou le nom) de la classe pour router vers le bon handler.

## Pourquoi utiliser un bus ?

1. **Évolutivité** : ajouter un nouveau comportement = ajouter un handler, sans toucher aux appels existants.
2. **Testabilité** : on peut tester les handlers isolément et mocker le bus dans les tests d’intégration.
3. **Cohérence** : transactions, logging et sécurité appliqués à tous les messages de la même façon.
4. **Alignement DDD/CQRS** : commandes et événements de domaine s’expriment naturellement comme messages ; le bus devient l’entrée principale du domaine.

## Résumé

- Un **bus applicatif** est un canal logiciel partagé pour envoyer des **messages** (commandes, événements) et les router vers les **handlers**.
- Il **découple** l’émetteur du récepteur, **centralise** le flux (middlewares) et facilite la **séparation** commandes/événements (CQRS, DDD).
- Les messages sont des **objets** (PHP) ; le bus utilise leur type pour déterminer quel handler les traite.

Dans la suite, nous détaillons les [concepts fondamentaux](./02-concepts-fondamentaux.md) : structure des messages, handlers, middleware et enveloppe.
