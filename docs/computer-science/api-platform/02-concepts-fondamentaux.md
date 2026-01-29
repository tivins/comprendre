# Concepts fondamentaux d'API Platform

## 1. La ressource (Resource)

Une **ressource** est l'unité de base exposée par l'API. En pratique, c'est une classe PHP marquée avec l'attribut `#[ApiResource]`. Chaque ressource correspond à un type d'objet métier (livre, utilisateur, commande, etc.).

### Déclaration minimale

```php
<?php
namespace App\Entity;

use ApiPlatform\Metadata\ApiResource;

#[ApiResource]
class Book
{
    public ?int $id = null;
    public string $title = '';
    public string $author = '';
}
```

À partir de cette classe, API Platform expose automatiquement :

- Une **collection** : `GET /books` (liste paginée)
- Un **item** : `GET /books/1` (détail)
- La **création** : `POST /books`
- La **mise à jour** : `PATCH /books/1`
- La **suppression** : `DELETE /books/1`

### Identifiant (IRI)

Chaque ressource est identifiée par un **IRI** (Internationalized Resource Identifier). En REST, c'est généralement l'URL : `/books/1`. L'IRI sert à référencer une ressource depuis une autre (ex. une revue qui pointe vers un livre via `"book": "/books/1"`).

## 2. Les opérations (Operations)

Une **opération** est une action HTTP exposée sur une ressource (Get, GetCollection, Post, Patch, Delete, etc.). Par défaut, API Platform active les opérations CRUD classiques ; vous pouvez en activer, désactiver ou personnaliser.

### Activer / désactiver des opérations

```php
#[ApiResource(
    operations: [
        new Get(),
        new GetCollection(),
        new Post(),
        new Patch(),
        new Delete(),
    ]
)]
class Book { ... }
```

Pour désactiver la suppression :

```php
#[ApiResource(
    operations: [
        new Get(),
        new GetCollection(),
        new Post(),
        new Patch(),
        // Delete désactivé
    ]
)]
class Book { ... }
```

### Opérations personnalisées

Vous pouvez ajouter des opérations custom (ex. `POST /books/{id}/publish`) en définissant une classe d'opération avec une route et un processeur dédié. Les concepts de **state provider** et **state processor** (voir plus bas) permettent d'y brancher votre logique.

## 3. La sérialisation (Serialization)

La **sérialisation** transforme les objets PHP en JSON (ou autre format) pour la réponse HTTP. La **désérialisation** fait l'inverse : corps de requête → objet PHP. API Platform s'appuie sur le composant **Serializer** de Symfony.

### Groupes de sérialisation

Pour exposer des champs différents selon le contexte (liste vs détail, public vs admin), on utilise des **groupes** :

```php
use Symfony\Component\Serializer\Annotation\Groups;

#[ApiResource(
    normalizationContext: ['groups' => ['book:read']],
    denormalizationContext: ['groups' => ['book:write']],
)]
class Book
{
    #[Groups(['book:read', 'book:write'])]
    public string $title = '';

    #[Groups(['book:read'])]  // lecture seule côté API
    public ?\DateTimeImmutable $createdAt = null;
}
```

- **normalizationContext** : utilisé à la sortie (PHP → JSON).
- **denormalizationContext** : utilisé à l'entrée (JSON → PHP).

### Relations et embeds

Les relations (ex. `Book` → `Review[]`) sont par défaut exposées comme des **IRI** (références) : `"reviews": ["/reviews/1", "/reviews/2"]`. Pour **embarquer** les objets liés dans la réponse, on utilise les groupes et la configuration de sérialisation (ou des normalizers personnalisés).

## 4. State providers et state processors

Pour **lire** et **écrire** les données, API Platform s'appuie sur deux concepts :

- **State provider** : fournit l'état d'une ressource (ou d'une collection). Il récupère les données (BDD, service externe, etc.).
- **State processor** : traite la création, la mise à jour ou la suppression d'une ressource (persistance, envoi d'événements, etc.).

### Approche « tout-en-un » (Doctrine)

Avec **Doctrine ORM**, un bridge fournit des state providers et processors par défaut : vos entités sont mappées en base, et API Platform lit/écrit via Doctrine. Vous n'avez pas à écrire vous-même les providers/processors pour un CRUD classique.

### Approche « séparation modèle public / persistance »

Si vous voulez séparer le **modèle exposé par l'API** du **modèle interne** (Clean Architecture, Hexagonal), vous définissez des **ressources** (DTO ou classes dédiées) et vous implémentez vous-même des **state providers** et **state processors** qui appellent vos services métier. API Platform reste agnostique de la persistance (BDD, NoSQL, API externe).

## 5. Persistance avec Doctrine (optionnel)

Quand vous utilisez Doctrine ORM, une ressource est typiquement une **entité** mappée avec les attributs Doctrine :

```php
use Doctrine\ORM\Mapping as ORM;

#[ORM\Entity]
#[ApiResource]
class Book
{
    #[ORM\Id, ORM\Column, ORM\GeneratedValue]
    private ?int $id = null;

    #[ORM\Column]
    public string $title = '';

    #[ORM\Column]
    public string $author = '';
}
```

Les state providers/processors Doctrine s'occupent alors de :

- **Get / GetCollection** : requêtes Doctrine (avec optimisation des JOIN si besoin).
- **Post / Patch / Delete** : persist, flush, remove.

Vous gardez la logique métier dans des **processors** personnalisés si besoin (validation métier, événements, etc.).

## 6. Résumé des concepts

| Concept | Rôle |
|--------|------|
| **Ressource** | Classe PHP `#[ApiResource]` = un type exposé par l'API (ex. Book, Review). |
| **Opération** | Action HTTP (Get, GetCollection, Post, Patch, Delete, ou custom). |
| **IRI** | Identifiant unique d'une ressource (souvent l'URL). |
| **Sérialisation** | PHP ↔ JSON (ou autre) avec groupes et contextes. |
| **State provider** | Récupère les données (lecture). |
| **State processor** | Traite création / mise à jour / suppression (écriture). |

Comprendre ces concepts permet d'expliquer simplement comment API Platform transforme des classes PHP en API REST (ou GraphQL) et où brancher sa logique (sérialisation, providers, processors).
