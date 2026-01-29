# Fonctionnalités avancées d'API Platform

## 1. Filtres et tri

### Filtres de recherche

API Platform fournit des **filtres** pour exposer des paramètres de requête (recherche, tri) sur les collections. Exemple avec un filtre de recherche sur le titre et l'auteur :

```php
use ApiPlatform\Metadata\ApiFilter;
use ApiPlatform\Doctrine\Orm\Filter\SearchFilter;

#[ApiResource]
#[ApiFilter(SearchFilter::class, properties: [
    'title' => 'partial',
    'author' => 'exact',
])]
class Book
{
    // ...
}
```

Résultat : `GET /books?title=doctrine&author=Kévin` interroge la collection avec ces critères.

### Tri (order)

Pour permettre le tri par propriété :

```php
use ApiPlatform\Doctrine\Orm\Filter\OrderFilter;

#[ApiFilter(OrderFilter::class, properties: ['title', 'publicationDate'])]
class Book { ... }
```

Exemple : `GET /books?order[title]=asc&order[publicationDate]=desc`.

### Filtres booléens et de plage

- **BooleanFilter** : filtre sur des champs booléens.
- **RangeFilter** : pour des nombres ou des dates (min/max).
- **DateFilter** : filtres avant/après une date.

Ces filtres se déclarent sur la ressource avec `#[ApiFilter(...)]` et des options (properties, stratégie).

## 2. Pagination

Les collections sont **paginées** par défaut (30 éléments par page, configurable). Le client reçoit des champs du type :

- `hydra:member` : tableau des éléments de la page.
- `hydra:totalItems` : nombre total d’éléments.
- `hydra:view` : liens `next` / `previous` pour la navigation.

Configuration globale (ex. dans `config/packages/api_platform.yaml`) :

```yaml
api_platform:
    defaults:
        pagination_enabled: true
        pagination_items_per_page: 30
```

Vous pouvez aussi définir une valeur par ressource via les attributs ou la configuration.

## 3. Validation des données

API Platform s’intègre au **composant Validator** de Symfony. Les contraintes déclarées sur la ressource (ou le DTO) sont exécutées avant d’appeler le state processor.

```php
use Symfony\Component\Validator\Constraints as Assert;

#[ApiResource]
class Book
{
    #[Assert\NotBlank]
    #[Assert\Length(min: 1, max: 200)]
    public string $title = '';

    #[Assert\Isbn]
    public ?string $isbn = null;

    #[Assert\Range(min: 0, max: 5)]
    public int $rating = 0;
}
```

En cas d’erreur, l’API renvoie une réponse **422 Unprocessable Entity** avec une structure type **Hydra** (ou RFC 7807) listant les violations (propriété, message). Côté client, ces erreurs sont faciles à afficher par champ.

## 4. Sécurité et contrôle d'accès

### Voters Symfony

Vous pouvez restreindre l’accès aux opérations ou aux instances avec les **voters** Symfony et l’attribut `security` :

```php
#[ApiResource(
    operations: [
        new Get(),
        new GetCollection(),
        new Post(security: "is_granted('ROLE_USER')"),
        new Patch(security: "is_granted('ROLE_ADMIN') or object.owner == user"),
        new Delete(security: "is_granted('ROLE_ADMIN')"),
    ]
)]
class Book { ... }
```

- **security** sur une opération : qui peut appeler cette action.
- **object** : la ressource concernée (pour Patch/Delete).
- **user** : l’utilisateur connecté (JWT, session, etc.).

### Sécurisation par ressource

Pour cacher des champs selon le rôle (ex. email uniquement pour l’admin), on combine **groupes de sérialisation** et **security** dans le contexte (ou des normalizers personnalisés) pour n’exposer certains groupes qu’à certains rôles.

## 5. GraphQL

API Platform peut exposer la **même** ressource en REST et en **GraphQL**. Il faut installer le composant GraphQL :

```bash
composer require api-platform/graphql
```

Les types, requêtes et mutations sont générés à partir des ressources. Le client peut alors :

- Faire des requêtes ciblées (seulement les champs nécessaires).
- Utiliser les filtres et la pagination (style Relay).
- Bénéficier de l’interface **GraphiQL** fournie pour tester.

Les règles de validation et de sécurité (voters) s’appliquent aussi en GraphQL.

## 6. Formats de sortie (content negotiation)

L’API peut servir plusieurs formats selon l’en-tête **Accept** ou l’extension d’URL :

- **JSON-LD** (défaut) : hypermédia, découvrabilité, Hydra.
- **JSON** : JSON simple.
- **JSON:API**, **HAL** : autres standards REST.
- **CSV**, **YAML** : possibles via des normalizers dédiés.

La **négociation de contenu** est configurable (formats par défaut, par ressource). Le format JSON-LD est souvent conservé par défaut pour l’admin et les clients qui s’appuient sur la doc Hydra.

## 7. Cache HTTP

API Platform propose un mécanisme de **cache HTTP** (invalidation) compatible avec Varnish ou reverse proxies. On peut marquer des réponses comme cachables et déclencher des invalidations quand une ressource est modifiée (via les tags ou les en-têtes). Utile pour les APIs à fort trafic en lecture.

## 8. Temps réel avec Mercure

Le protocole **Mercure** permet d’envoyer des mises à jour en temps réel aux clients (Server-Sent Events). API Platform peut publier des événements Mercure lors de la création/mise à jour/suppression de ressources. Les clients s’abonnent à des topics (ex. `/books/1`) et reçoivent les changements sans polling.

## 9. Résumé

| Fonctionnalité | Rôle |
|----------------|------|
| **Filtres** | Recherche, tri, plages (SearchFilter, OrderFilter, etc.). |
| **Pagination** | Collections découpées en pages avec liens next/previous. |
| **Validation** | Contraintes Symfony Validator, réponses 422 structurées. |
| **Sécurité** | Voters + `security` sur les opérations et champs. |
| **GraphQL** | Même modèle exposé en REST et GraphQL. |
| **Formats** | JSON-LD, JSON, JSON:API, HAL, etc. |
| **Cache** | Cache HTTP et invalidation (Varnish). |
| **Mercure** | Temps réel (SSE) sur les ressources. |

Ces briques permettent d’aller au-delà du CRUD minimal et de construire des APIs robustes, sécurisées et adaptées à des clients variés (web, mobile, open data).
