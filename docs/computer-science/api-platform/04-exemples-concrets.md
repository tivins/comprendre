# Exemples concrets avec API Platform

## Contexte : API Bookshop

On construit une petite API de librairie avec deux types de ressources :

- **Book** : livre (ISBN, titre, description, auteur, date de publication, liste d’avis).
- **Review** : avis sur un livre (note 0–5, texte, auteur, date, livre associé).

Les exemples suivants illustrent les concepts vus dans les chapitres précédents.

## 1. Ressources et entités Doctrine

### Entité Book

```php
<?php
namespace App\Entity;

use ApiPlatform\Metadata\ApiResource;
use ApiPlatform\Metadata\ApiFilter;
use ApiPlatform\Doctrine\Orm\Filter\SearchFilter;
use ApiPlatform\Doctrine\Orm\Filter\OrderFilter;
use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\ORM\Mapping as ORM;
use Symfony\Component\Validator\Constraints as Assert;

#[ORM\Entity]
#[ApiResource]
#[ApiFilter(SearchFilter::class, properties: ['title' => 'partial', 'author' => 'partial'])]
#[ApiFilter(OrderFilter::class, properties: ['title', 'publicationDate'])]
class Book
{
    #[ORM\Id, ORM\Column, ORM\GeneratedValue]
    private ?int $id = null;

    #[ORM\Column(nullable: true)]
    #[Assert\Isbn]
    public ?string $isbn = null;

    #[ORM\Column]
    #[Assert\NotBlank]
    public string $title = '';

    #[ORM\Column(type: 'text')]
    #[Assert\NotBlank]
    public string $description = '';

    #[ORM\Column]
    #[Assert\NotBlank]
    public string $author = '';

    #[ORM\Column]
    #[Assert\NotNull]
    public ?\DateTimeImmutable $publicationDate = null;

    /** @var Review[] */
    #[ORM\OneToMany(targetEntity: Review::class, mappedBy: 'book', cascade: ['persist', 'remove'])]
    public iterable $reviews;

    public function __construct()
    {
        $this->reviews = new ArrayCollection();
    }

    public function getId(): ?int
    {
        return $this->id;
    }
}
```

### Entité Review

```php
<?php
namespace App\Entity;

use ApiPlatform\Metadata\ApiResource;
use Doctrine\ORM\Mapping as ORM;
use Symfony\Component\Validator\Constraints as Assert;

#[ORM\Entity]
#[ApiResource]
class Review
{
    #[ORM\Id, ORM\Column, ORM\GeneratedValue]
    private ?int $id = null;

    #[ORM\Column(type: 'smallint')]
    #[Assert\Range(min: 0, max: 5)]
    public int $rating = 0;

    #[ORM\Column(type: 'text')]
    #[Assert\NotBlank]
    public string $body = '';

    #[ORM\Column]
    #[Assert\NotBlank]
    public string $author = '';

    #[ORM\Column]
    #[Assert\NotNull]
    public ?\DateTimeImmutable $publicationDate = null;

    #[ORM\ManyToOne(inversedBy: 'reviews')]
    #[Assert\NotNull]
    public ?Book $book = null;

    public function getId(): ?int
    {
        return $this->id;
    }
}
```

Avec ces deux classes, API Platform expose :

- `GET /books`, `GET /books/{id}`, `POST /books`, `PATCH /books/{id}`, `DELETE /books/{id}`.
- `GET /reviews`, `GET /reviews/{id}`, `POST /reviews`, `PATCH /reviews/{id}`, `DELETE /reviews/{id}`.
- Filtres : `?title=doctrine`, `?author=Kévin`, `?order[title]=asc`.
- Validation : ISBN, champs obligatoires, note entre 0 et 5.

## 2. Créer un livre (POST)

Requête :

```http
POST /books
Content-Type: application/ld+json

{
  "isbn": "9781782164104",
  "title": "Persistence in PHP with the Doctrine ORM",
  "description": "A book for PHP developers and architects.",
  "author": "Kévin Dunglas",
  "publicationDate": "2013-12-01"
}
```

Réponse typique (201 Created) : corps JSON-LD avec `@id`, `@type`, et les champs du livre. L’IRI du livre (ex. `/books/1`) sert à référencer ce livre dans d’autres ressources (ex. une Review).

## 3. Créer un avis lié à un livre (POST)

Requête :

```http
POST /reviews
Content-Type: application/ld+json

{
  "book": "/books/1",
  "rating": 5,
  "body": "Interesting book!",
  "author": "Kévin",
  "publicationDate": "2016-09-21"
}
```

Le champ **book** est une **référence IRI** : le client envoie l’IRI du livre. API Platform résout l’IRI vers l’entité `Book` et associe la revue au bon livre. C’est le comportement standard pour les relations en hypermédia.

## 4. Erreur de validation (422)

Requête invalide (titre vide, ISBN invalide) :

```http
POST /books
Content-Type: application/ld+json

{
  "isbn": "2815840053",
  "description": "Hello",
  "author": "Me",
  "publicationDate": "today"
}
```

Réponse **422 Unprocessable Entity** (format type Hydra) :

```json
{
  "@context": "/contexts/ConstraintViolationList",
  "@type": "ConstraintViolationList",
  "title": "An error occurred",
  "violations": [
    { "propertyPath": "isbn", "message": "This value is neither a valid ISBN-10 nor a valid ISBN-13." },
    { "propertyPath": "title", "message": "This value should not be blank." }
  ]
}
```

Le client peut afficher les messages par champ (`propertyPath` / `message`).

## 5. Groupes de sérialisation (liste vs détail)

Pour exposer moins de champs en liste qu’en détail :

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

    #[Groups(['book:read', 'book:write'])]
    public string $author = '';

    #[Groups(['book:read'])]  // seulement en lecture (réponse)
    public ?\DateTimeImmutable $publicationDate = null;

    #[Groups(['book:item'])]  // seulement pour GET /books/{id}
    public iterable $reviews;
}
```

En configurant des groupes différents pour la collection et l’item (via les opérations), on obtient une liste légère et un détail riche (avec les avis).

## 6. Restreindre les opérations

Exemple : pas de suppression pour les livres, création d’avis réservée aux utilisateurs authentifiés :

```php
use ApiPlatform\Metadata\Get;
use ApiPlatform\Metadata\GetCollection;
use ApiPlatform\Metadata\Post;
use ApiPlatform\Metadata\Patch;

#[ApiResource(
    operations: [
        new Get(),
        new GetCollection(),
        new Post(security: "is_granted('ROLE_USER')"),
        new Patch(security: "is_granted('ROLE_USER')"),
        // Pas de Delete
    ]
)]
class Book { ... }
```

Même principe pour `Review` : on peut exiger `ROLE_USER` sur Post/Patch et laisser Get/GetCollection publics.

## 7. Résumé des cas couverts

| Cas | Méthode |
|-----|--------|
| Exposer une ressource | `#[ApiResource]` + entité Doctrine (ou DTO + state providers/processors). |
| Filtres et tri | `#[ApiFilter(SearchFilter::class, ...)]`, `OrderFilter`, etc. |
| Validation | Contraintes `#[Assert\*]` sur les propriétés. |
| Relations | Référence par IRI dans le JSON (`"book": "/books/1"`). |
| Champs conditionnels | Groupes de sérialisation + contextes. |
| Sécurité | `security` sur les opérations, voters Symfony. |

Ces exemples montrent comment, en peu de code PHP, on obtient une API REST documentée, validée, filtrée et sécurisable, prête à être consommée par un front ou une app mobile.
