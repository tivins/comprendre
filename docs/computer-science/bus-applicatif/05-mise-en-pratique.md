# Mise en pratique : bus applicatif

## 1. Quand introduire un bus ?

### 1.1 Indications favorables

- **Découplage** : vous voulez que le contrôleur (ou l’API) ne dépende pas directement des services métier ; une seule entrée (le bus) pour toutes les actions.
- **CQRS / DDD** : vous modélisez des commandes et des événements de domaine ; un bus permet de les dispatcher proprement (command bus + event bus).
- **Comportements transversaux** : vous avez besoin de transactions, logging, validation ou retry de façon uniforme pour toutes les commandes (middlewares).
- **Évolutivité** : vous prévoyez d’ajouter souvent de nouveaux cas d’usage ou de nouvelles réactions (handlers) sans modifier les points d’entrée.

### 1.2 Quand s’en passer

- **Application très simple** : peu de cas d’usage, pas de CQRS ni d’événements ; un appel direct au service peut suffire.
- **Équipe ou projet minimal** : éviter la complexité inutile ; le bus apporte de la structure mais aussi des concepts à maîtriser (messages, handlers, transports).

## 2. Choix d’implémentation (PHP)

### 2.1 Symfony Messenger

- **Avantages** : intégré à Symfony, middlewares (validation, transaction), transports (Doctrine, Redis, AMQP), attributs `#[AsMessageHandler]`, enveloppes et stamps.
- **Usage typique** : projet Symfony ou API Symfony ; besoin de synchrone et asynchrone, CQRS, DDD.

### 2.2 Tactician (League)

- **Avantages** : léger, indépendant du framework, orienté command bus, middlewares par pipeline.
- **Usage typique** : projet PHP sans Symfony ou besoin d’un command bus simple sans event bus intégré.

### 2.3 Implémentation minimale

Pour comprendre le principe, on peut implémenter un bus minimal : un tableau « type de message → handler », une boucle de middlewares, et un `dispatch($message)` qui appelle les middlewares puis le handler. Utile en formation ou en prototype.

## 3. Bonnes pratiques

### 3.1 Messages

- **Immuables** : messages en `readonly` (PHP 8+) et sans setters pour éviter les modifications après création.
- **Porteurs de données uniquement** : pas de logique métier dans le message ; le handler contient l’orchestration et appelle le domaine.
- **Nommage** : commandes à l’impératif (`CreateOrder`), événements au passé (`OrderCreated`), cohérent avec le langage ubiquitaire (DDD).

### 3.2 Handlers

- **Un handler par type de message** pour les commandes ; pour les événements, autant de handlers que de réactions (notification, projection, etc.).
- **Handlers fins** : un handler fait une chose ; s’il grossit, extraire des services ou sous-handlers.
- **Pas d’envoi de commandes depuis un handler d’événement** sans réflexion : risque de boucles ou de flux difficiles à suivre ; privilégier des événements en chaîne ou des saga si besoin.

### 3.3 Middlewares

- **Ordre** : validation → transaction → (autres) → handler ; logging autour de tout.
- **Transaction** : un middleware qui ouvre une transaction avant le handler et committe/rollback après évite de dupliquer le code dans chaque handler.
- **Idempotence** : pour l’asynchrone, concevoir les handlers pour qu’un double traitement (retry) ne crée pas d’effet de bord en double (ex. utiliser un ID de message ou une clé métier).

### 3.4 Tests

- **Handlers** : tester chaque handler isolément en lui passant un message ; mocker les dépendances (repository, event bus).
- **Bus** : tests d’intégration qui envoient un message sur le bus et vérifient le résultat ou les effets (persistence, événements publiés).
- **Contrôleurs** : tester que le contrôleur envoie le bon message au bus et renvoie la bonne réponse ; le bus peut être mocké pour ne pas exécuter les vrais handlers.

## 4. Pièges à éviter

### 4.1 Mélanger commandes et requêtes sur le même bus

En CQRS, les **requêtes** (lecture) n’ont pas besoin de passer par le command bus. On peut avoir des query handlers ou des services de lecture appelés directement depuis le contrôleur pour éviter la surcharge conceptuelle.

### 4.2 Handlers trop gros

Un handler qui fait création + envoi d’email + mise à jour de 3 projections devient illisible et difficile à tester. Mieux vaut : le handler de commande qui crée l’agrégat et publie un événement ; des handlers d’événement distincts pour email, projection 1, projection 2, etc.

### 4.3 Événements asynchrones et cohérence

En asynchrone, l’utilisateur ne voit pas immédiatement le résultat des handlers d’événement (ex. projection). Il faut accepter une **cohérence à terme** (eventual consistency) ou prévoir un mécanisme de rafraîchissement (polling, WebSocket) pour la lecture.

### 4.4 Débogage

Avec un bus, le flux n’est plus un simple appel de méthode. Utiliser le **logging** (middleware) pour tracer les messages entrés/sortis et les erreurs ; en Symfony, les stamps et le profiler Messenger aident à suivre les messages et les transports.

## 5. Intégration progressive

- **Étape 1** : Introduire un **command bus** pour une ou deux commandes pilotes (ex. `CreateOrder`). Le contrôleur envoie la commande ; le handler appelle le service ou l’agrégat existant.
- **Étape 2** : Ajouter des **middlewares** (transaction, validation, logging) pour uniformiser le comportement.
- **Étape 3** : Introduire un **event bus** : le handler de commande publie un événement ; un ou deux handlers d’événement (ex. email, projection) réagissent. Étendre progressivement les événements et les handlers.
- **Étape 4** : Si besoin, passer certains événements en **asynchrone** (transport + workers) pour alléger la requête HTTP et améliorer la résilience.

## Résumé

- Introduire un bus quand le **découplage**, le **CQRS/DDD** ou les **middlewares** apportent un bénéfice clair.
- En PHP, **Symfony Messenger** convient bien pour un projet Symfony avec commandes, événements et asynchrone ; **Tactician** pour un command bus léger.
- Bonnes pratiques : messages **immuables** et **nommés** (langage ubiquitaire), handlers **fins**, middlewares pour **transaction** et **logging**, conception **idempotente** pour l’asynchrone.
- Éviter : mélanger requêtes et commandes sur le même bus sans besoin, handlers trop gros, oubli de la cohérence à terme en asynchrone.
- Intégration **progressive** : command bus d’abord, puis middlewares, puis event bus, puis asynchrone si nécessaire.

---

Retour au [sommaire de la documentation](./README.md).
