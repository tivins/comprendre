# Exemples concrets

Ce chapitre illustre les concepts et stratégies sur des cas réalistes : extraction d’un service avec introduction d’un seam, ajout de tests de caractérisation sur du code non testé, et remplacement d’une dépendance (branche par abstraction). Les exemples sont en pseudo-code inspiré de Java/TypeScript/PHP.

## Exemple 1 : Introduire un seam pour tester et remplacer

### Contexte

Une classe `OrderProcessor` calcule le total d’une commande et envoie un email de confirmation. Elle instancie directement un `Mailer` et accède à la BDD via une connexion globale. On ne peut pas tester la logique de calcul sans envoyer de vrais emails ni toucher à la BDD.

### Code initial (simplifié)

```php
class OrderProcessor {
    public function process(Order $order): void {
        $total = $this->computeTotal($order);
        $order->setTotal($total);
        Database::getConnection()->save($order);
        $mailer = new Mailer();
        $mailer->send($order->getUser()->getEmail(), 'Order confirmed', 'Total: ' . $total);
    }

    private function computeTotal(Order $order): float {
        $sum = 0;
        foreach ($order->getLines() as $line) {
            $sum += $line->getQuantity() * $line->getUnitPrice();
        }
        return $sum;
    }
}
```

### Introduction de seams (interface + injection)

On extrait les dépendances derrière des interfaces et on les injecte au constructeur. Le comportement métier (calcul, envoi, persistance) ne change pas ; on crée des **points de couture** pour pouvoir mocker en test et remplacer plus tard.

```php
interface OrderRepository {
    public function save(Order $order): void;
}

interface MailerInterface {
    public function send(string $to, string $subject, string $body): void;
}

class OrderProcessor {
    public function __construct(
        private OrderRepository $repository,
        private MailerInterface $mailer
    ) {}

    public function process(Order $order): void {
        $total = $this->computeTotal($order);
        $order->setTotal($total);
        $this->repository->save($order);
        $this->mailer->send(
            $order->getUser()->getEmail(),
            'Order confirmed',
            'Total: ' . $total
        );
    }

    private function computeTotal(Order $order): float {
        $sum = 0;
        foreach ($order->getLines() as $line) {
            $sum += $line->getQuantity() * $line->getUnitPrice();
        }
        return $sum;
    }
}
```

En test : on injecte un `OrderRepository` mock (en mémoire ou stub) et un `MailerInterface` mock (qui n’envoie rien). On appelle `process()` et on vérifie le total calculé et les appels au repository et au mailer. La logique métier est désormais testable sans BDD ni email réel.

---

## Exemple 2 : Tests de caractérisation sur du code non testé

### Contexte

Une fonction `formatInvoiceReference(Order $order): string` est utilisée partout pour afficher une référence facture. Personne ne sait exactement quelles règles (dates, IDs, préfixes) sont appliquées ; il n’y a pas de tests. Avant de refactoriser, on ajoute des tests de caractérisation.

### Démarche

1. Choisir quelques cas représentatifs : commande simple, commande avec date à minuit, commande avec ID à plusieurs chiffres, etc.
2. Exécuter la fonction avec ces entrées et noter le résultat actuel.
3. Écrire un test qui appelle la fonction et affirme que le résultat est égal à la valeur observée.

```java
@Test
void formatInvoiceReference_characterization_simpleOrder() {
    Order order = anOrder().withId(42).withDate(LocalDate.of(2024, 3, 15)).build();
    String result = LegacyInvoiceFormatter.formatInvoiceReference(order);
    // Valeur observée au moment de l’écriture du test ; documente le comportement actuel
    assertEquals("INV-2024-03-00042", result);
}
```

Si plus tard quelqu’un refactore et change le format, le test échoue. L’équipe décide alors soit de corriger le refactoring, soit de mettre à jour le test (et la doc) si le nouveau comportement est voulu. Sans ce test, le refactoring aurait pu casser des factures sans que personne ne le voie tout de suite.

---

## Exemple 3 : Branche par abstraction — remplacer un service de paiement

### Contexte

L’application utilise `LegacyPaymentGateway` (SDK propriétaire, difficile à tester, déprécié). On veut passer à un nouveau fournisseur `NewPaymentGateway` sans interruption. On applique « branche par abstraction ».

### Étape 1 : Définir l’interface

On extrait le contrat dont ont besoin les appelants (par ex. « charger un paiement » et « obtenir le statut »).

```typescript
interface PaymentGateway {
    charge(amount: number, currency: string, token: string): Promise<ChargeResult>;
    getStatus(chargeId: string): Promise<ChargeStatus>;
}
```

### Étape 2 : Implémentations

`LegacyPaymentGatewayAdapter` encapsule l’ancien SDK et implémente `PaymentGateway`. `NewPaymentGatewayAdapter` encapsule le nouveau fournisseur et implémente la même interface. Les deux sont testables : l’ancien via des tests de caractérisation (comportement actuel), le nouveau via des tests sur le contrat.

### Étape 3 : Bascule

Une factory ou le conteneur d’injection retourne l’une ou l’autre implémentation selon la config (variable d’environnement, feature flag). On déploie d’abord avec l’ancienne ; on active la nouvelle pour un pourcentage du trafic ou un environnement de test ; on généralise ; on supprime l’ancienne implémentation et l’adapter legacy.

Les appelants ne changent pas : ils dépendent de `PaymentGateway`. Seul le choix de l’implémentation change.

---

## Synthèse

| Exemple | Concept / stratégie | Bénéfice |
|---------|---------------------|----------|
| Seam sur `OrderProcessor` | Seams, injection de dépendances | Tester la logique sans BDD ni mail ; préparer un remplacement |
| Tests sur `formatInvoiceReference` | Tests de caractérisation | Documenter le comportement actuel avant tout refactoring |
| Remplacement `PaymentGateway` | Branche par abstraction | Changer d’implémentation sans toucher aux appelants ; bascule progressive |

---

## Prochaines étapes

- **[Mise en pratique](./05-mise-en-pratique.md)** : quand refactoriser, pièges courants, intégration dans le workflow
