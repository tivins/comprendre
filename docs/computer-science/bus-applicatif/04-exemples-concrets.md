# Exemples concrets : bus applicatif en PHP avec DDD

Cette section présente des exemples concrets d’utilisation d’un bus applicatif en PHP, en lien avec le Domain-Driven Design (agrégats, événements de domaine, CQRS).

## 1. Contexte : application de commandes (e-commerce)

### 1.1 Domaine métier

- Un **client** peut passer une **commande** (panier d’articles, adresse de livraison).
- Une commande est créée avec un statut « en attente » ; elle peut être confirmée, expédiée, annulée.
- Lors de la création, on réserve le stock, on envoie un email de confirmation et on met à jour une projection pour l’affichage.

### 1.2 Rôle du bus

- **Command bus** : la commande métier « Créer une commande » est envoyée comme message `CreateOrder` ; un handler unique crée l’agrégat `Order` et publie l’événement `OrderCreated`.
- **Event bus** : l’événement `OrderCreated` est traité par plusieurs handlers : envoi d’email, mise à jour de la projection, réservation du stock (ou envoi d’une commande vers le service stock).

## 2. Messages : commandes et événements (PHP 8+)

### 2.1 Commande : CreateOrder

```php
namespace App\Order\Application\Command;

final readonly class CreateOrder
{
    /**
     * @param array<int, array{productId: string, quantity: int}> $items
     */
    public function __construct(
        public string $customerId,
        public array $items,
        public string $deliveryAddress,
    ) {}
}
```

### 2.2 Événement : OrderCreated

```php
namespace App\Order\Application\Event;

final readonly class OrderCreated
{
    public function __construct(
        public string $orderId,
        public string $customerId,
        public \DateTimeImmutable $createdAt,
    ) {}
}
```

## 3. Agrégat Order (domaine)

L’agrégat encapsule les règles métier et émet des événements de domaine (ou le handler les construit à partir de l’agrégat).

```php
namespace App\Order\Domain;

final class Order
{
    private function __construct(
        private string $id,
        private string $customerId,
        /** @var array<int, OrderItem> */
        private array $items,
        private string $deliveryAddress,
        private \DateTimeImmutable $createdAt,
        private OrderStatus $status,
    ) {}

    public static function create(
        string $customerId,
        array $items,
        string $deliveryAddress,
    ): self {
        $id = uniqid('order_', true);
        $orderItems = array_map(
            fn (array $item) => new OrderItem(
                $item['productId'],
                $item['quantity'],
            ),
            $items,
        );
        return new self(
            $id,
            $customerId,
            $orderItems,
            $deliveryAddress,
            new \DateTimeImmutable(),
            OrderStatus::Pending,
        );
    }

    public function id(): string { return $this->id; }
    public function customerId(): string { return $this->customerId; }
    public function createdAt(): \DateTimeImmutable { return $this->createdAt; }
}
```

## 4. Handler de la commande CreateOrder

Le handler reçoit la commande, crée l’agrégat, le persiste et publie l’événement sur l’event bus.

```php
namespace App\Order\Application\Command;

use App\Order\Application\Event\OrderCreated;
use App\Order\Domain\Order;
use App\Order\Domain\OrderRepositoryInterface;
use Symfony\Component\Messenger\Attribute\AsMessageHandler;
use Symfony\Component\Messenger\MessageBusInterface;

#[AsMessageHandler]
final readonly class CreateOrderHandler
{
    public function __construct(
        private OrderRepositoryInterface $orderRepository,
        private MessageBusInterface $eventBus,
    ) {}

    public function __invoke(CreateOrder $command): string
    {
        $order = Order::create(
            $command->customerId,
            $command->items,
            $command->deliveryAddress,
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

## 5. Handlers de l’événement OrderCreated

Plusieurs handlers réagissent au même événement ; chacun fait une chose (notification, projection, intégration).

### 5.1 Envoi d’email de confirmation

```php
namespace App\Order\Application\Event;

use App\Notification\Service\OrderConfirmationMailer;
use Symfony\Component\Messenger\Attribute\AsMessageHandler;

#[AsMessageHandler]
final readonly class SendOrderConfirmationHandler
{
    public function __construct(
        private OrderConfirmationMailer $mailer,
    ) {}

    public function __invoke(OrderCreated $event): void
    {
        $this->mailer->send($event->orderId, $event->customerId);
    }
}
```

### 5.2 Mise à jour de la projection (lecture CQRS)

```php
namespace App\Order\Application\Event;

use App\Order\ReadModel\OrderProjectionRepository;
use Symfony\Component\Messenger\Attribute\AsMessageHandler;

#[AsMessageHandler]
final readonly class UpdateOrderProjectionHandler
{
    public function __construct(
        private OrderProjectionRepository $projectionRepository,
    ) {}

    public function __invoke(OrderCreated $event): void
    {
        $this->projectionRepository->addOrder(
            $event->orderId,
            $event->customerId,
            $event->createdAt,
        );
    }
}
```

## 6. Contrôleur : envoi sur le bus

Le contrôleur ne connaît que le bus ; il ne dépend pas des handlers ni des services métier.

```php
namespace App\Order\Interface\Http;

use App\Order\Application\Command\CreateOrder;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Messenger\MessageBusInterface;
use Symfony\Component\Messenger\Stamp\HandledStamp;
use Symfony\Component\Routing\Annotation\Route;

final class CreateOrderController
{
    public function __construct(
        private MessageBusInterface $commandBus,
    ) {}

    #[Route('/api/orders', methods: ['POST'])]
    public function __invoke(Request $request): Response
    {
        $data = $request->toArray();
        $command = new CreateOrder(
            $data['customerId'],
            $data['items'],
            $data['deliveryAddress'],
        );
        $envelope = $this->commandBus->dispatch($command);
        $handledStamp = $envelope->last(HandledStamp::class);
        $orderId = $handledStamp?->getResult();
        return new Response(json_encode(['orderId' => $orderId]), 201, [
            'Content-Type' => 'application/json',
        ]);
    }
}
```

## 7. Configuration Symfony Messenger (exemple)

On configure deux bus : un pour les commandes (synchrone), un pour les événements (éventuellement asynchrone).

```yaml
# config/packages/messenger.yaml
framework:
    messenger:
        default_bus: command.bus
        buses:
            command.bus:
                middleware:
                    - validation
                    - doctrine_transaction
            event.bus:
                middleware:
                    - validation
        transports:
            async_events:
                dsn: '%env(MESSENGER_TRANSPORT_DSN)%'
                options:
                    queue_name: events
        routing:
            App\Order\Application\Event\OrderCreated: async_events
```

Les commandes partent sur `command.bus` (synchrone par défaut) ; les événements comme `OrderCreated` sont routés vers le transport `async_events` pour être traités par des workers.

## 8. Résumé des flux

| Étape | Acteur | Message | Bus | Handler(s) |
|-------|--------|---------|-----|------------|
| 1 | Contrôleur | `CreateOrder` | command.bus | `CreateOrderHandler` |
| 2 | CreateOrderHandler | `OrderCreated` | event.bus | `SendOrderConfirmationHandler`, `UpdateOrderProjectionHandler`, … |

Le bus applicatif permet ainsi de découpler le contrôleur du domaine, et le domaine des réactions (notifications, projections) en s’appuyant sur des messages (commandes et événements) et des handlers dédiés.

Pour aller plus loin, voir la [mise en pratique](./05-mise-en-pratique.md) : quand introduire un bus, choix d’implémentation (Symfony Messenger, Tactician), bonnes pratiques et pièges à éviter.
