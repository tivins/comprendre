# Concepts fondamentaux du bus applicatif

## 1. Le message

Un **message** est l’unité d’échange sur le bus. Il porte une **intention** (commande) ou un **fait** (événement) et les données associées.

### 1.1 Structure d’un message

En PHP, un message est typiquement une **classe** (objet) avec des propriétés. Le bus utilise le **type** (nom de la classe) pour router le message vers le bon handler.

**Exemple : commande**

```php
final readonly class CreateOrder
{
    public function __construct(
        public string $customerId,
        /** @var array<array{productId: string, quantity: int}> */
        public array $items,
        public string $deliveryAddress,
    ) {}
}
```

**Exemple : événement**

```php
final readonly class OrderCreated
{
    public function __construct(
        public string $orderId,
        public string $customerId,
        public \DateTimeImmutable $createdAt,
    ) {}
}
```

### 1.2 Bonnes pratiques pour les messages

- **Immuables** : une fois créé, un message ne doit pas être modifié (en PHP : `readonly` sur la classe et les propriétés).
- **Porteurs de données** : pas de logique métier dans le message, uniquement des données.
- **Nommage explicite** : commandes à l’impératif (`CreateOrder`, `CancelSubscription`), événements au passé (`OrderCreated`, `UserRegistered`).

## 2. Le handler

Un **handler** est le composant qui **traite** un type de message donné. Le bus appelle le handler correspondant au type du message.

### 2.1 Convention : un handler par type de message (commandes)

Pour une **commande**, on a en général **un seul** handler. En PHP, l’interface peut ressembler à :

```php
interface MessageHandlerInterface
{
    public function __invoke(object $message): mixed;
}
```

Exemple de handler pour la commande `CreateOrder` :

```php
final readonly class CreateOrderHandler implements MessageHandlerInterface
{
    public function __construct(
        private OrderRepository $orderRepository,
        private EventBus $eventBus,
    ) {}

    public function __invoke(object $message): mixed
    {
        if (!$message instanceof CreateOrder) {
            throw new \InvalidArgumentException('Expected CreateOrder');
        }

        $order = Order::create(
            $message->customerId,
            $message->items,
            $message->deliveryAddress,
        );
        $this->orderRepository->save($order);
        $this->eventBus->dispatch(new OrderCreated(
            $order->id(),
            $order->customerId(),
            $order->createdAt(),
        ));
        return $order->id();
    }
}
```

### 2.2 Plusieurs handlers pour un même message (événements)

Pour un **événement**, le bus peut appeler **plusieurs** handlers (notifications, mises à jour de projections, envoi à un autre système). Chaque handler est indépendant ; l’ordre d’exécution peut être défini par configuration ou par priorité.

## 3. Le middleware

Un **middleware** est une couche exécutée **avant** et **après** le traitement du message par le handler. Il permet de factoriser la logique transversale sans la dupliquer dans chaque handler.

### 3.1 Rôle typique des middlewares

| Middleware   | Rôle |
|-------------|------|
| **Transaction** | Démarrer une transaction avant le handler, committer en cas de succès, rollback en cas d’exception |
| **Logging**     | Logger l’entrée (type du message, id) et la sortie (durée, succès/échec) |
| **Validation**  | Valider le message (données, contraintes métier) avant d’appeler le handler |
| **Sécurité**    | Vérifier que l’utilisateur courant a le droit d’exécuter cette commande |
| **Retry**       | En cas d’exception temporaire, réessayer selon une stratégie (backoff, nombre max) |

### 3.2 Forme d’un middleware (pseudo-code)

Le middleware reçoit le message et une « next » qui représente la chaîne restante (handler ou middleware suivant). Il peut faire des actions avant et après l’appel à `next` :

```php
interface MiddlewareInterface
{
    public function handle(object $message, callable $next): mixed;
}

final class TransactionMiddleware implements MiddlewareInterface
{
    public function __construct(private Connection $connection) {}

    public function handle(object $message, callable $next): mixed
    {
        $this->connection->beginTransaction();
        try {
            $result = $next($message);
            $this->connection->commit();
            return $result;
        } catch (\Throwable $e) {
            $this->connection->rollBack();
            throw $e;
        }
    }
}
```

L’ordre des middlewares est important : par exemple, validation avant transaction, logging autour de tout.

## 4. Enveloppe et métadonnées

Parfois, le message est enveloppé dans un objet qui porte des **métadonnées** (ID de message, timestamp, trace, utilisateur) en plus du message « nu ». Le handler reçoit alors soit l’enveloppe (et en extrait le message), soit directement le message si le bus déplie l’enveloppe avant d’appeler le handler.

**Exemple d’enveloppe (conceptuel)** :

```php
final readonly class Envelope
{
    public function __construct(
        public object $message,
        public array $stamps = [], // e.g. BusNameStamp, SerializerContextStamp
    ) {}
}
```

Les « stamps » permettent d’ajouter des informations pour le transport (nom de la file, retry, etc.) sans modifier le message métier.

## 5. Routage : message → handler

Le bus doit savoir **quel handler** appeler pour **quel type** de message. Ce routage peut être :

- **Convention** : une classe `CreateOrder` est gérée par `CreateOrderHandler` (même nom + suffixe `Handler`).
- **Configuration** : un tableau ou un fichier (YAML, PHP) qui associe `CreateOrder::class` à `CreateOrderHandler::class`.
- **Attributs / annotations** (PHP 8) : `#[AsMessageHandler]` sur la classe du handler, avec le type du message en paramètre.

Exemple avec attribut (Symfony Messenger) :

```php
use Symfony\Component\Messenger\Attribute\AsMessageHandler;

#[AsMessageHandler]
final readonly class CreateOrderHandler
{
    public function __invoke(CreateOrder $command): string
    {
        // ...
        return $order->id();
    }
}
```

Le bus résout alors automatiquement que `CreateOrder` est géré par `CreateOrderHandler`.

## 6. Synchrone vs asynchrone

- **Synchrone** : l’émetteur envoie le message et attend la fin du traitement ; le handler s’exécute dans le même processus (et souvent le même thread/request).
- **Asynchrone** : l’émetteur envoie le message sur une **file** (queue) ; un **worker** (processus ou cron) consomme la file et appelle le handler. L’émetteur ne bloque pas ; le résultat n’est pas disponible immédiatement (sauf si on utilise un mécanisme type « job + polling » ou « async + callback »).

En PHP, le traitement asynchrone repose en général sur un transport (Doctrine DBAL, Redis, AMQP) et des workers (Symfony Messenger avec `messenger:consume`, ou équivalent).

## Résumé

- Un **message** est un objet (classe PHP) porteur de données ; **commande** = intention (un handler), **événement** = fait (zéro à N handlers).
- Un **handler** traite un type de message ; il est invoqué par le bus après routage.
- Un **middleware** encapsule le traitement (transaction, logging, validation, retry) sans dupliquer le code dans chaque handler.
- **Enveloppe** et **stamps** permettent d’ajouter des métadonnées au message pour le transport et le traitement.
- Le **routage** associe un type de message à un (ou plusieurs) handler(s) par convention, configuration ou attributs.
- Le bus peut être **synchrone** (traitement immédiat) ou **asynchrone** (file + workers).

Dans la suite, nous détaillons les [types et usages](./03-types-et-usage.md) : command bus, event bus et intégration avec le DDD.
