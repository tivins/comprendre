# Introduction à API Platform

## Qu'est-ce qu'API Platform ?

**API Platform** est un framework full stack pour créer des APIs web. L'idée centrale : vous définissez votre modèle de données sous forme de classes PHP (entités ou DTO), et le framework expose automatiquement une API REST (et optionnellement GraphQL) avec documentation, pagination, filtres et sérialisation.

### Une API à partir de classes

Au lieu d'écrire à la main chaque route, contrôleur et schéma OpenAPI, vous déclarez une **ressource** (une classe) et API Platform génère :

- Les opérations CRUD (Create, Read, Update, Delete)
- La documentation OpenAPI / Swagger UI
- La sérialisation JSON (ou JSON-LD, JSON:API, HAL, etc.)
- La pagination et les filtres de base

### Analogie rapide

Imaginez un **ORM pour les APIs** : comme Doctrine mappe des classes PHP vers des tables SQL, API Platform mappe des classes PHP vers des ressources HTTP. Vous travaillez en objets ; le framework s'occupe des URLs, des méthodes HTTP et des formats d'échange.

## Pourquoi utiliser API Platform ?

### Avantages

1. **Gain de temps** : une classe marquée `#[ApiResource]` suffit pour avoir une API opérationnelle.
2. **Standards** : support natif de JSON-LD/Hydra, OpenAPI, JSON:API, HAL, GraphQL.
3. **Documentation automatique** : Swagger UI et GraphiQL générés à partir du code.
4. **Écosystème cohérent** : admin dynamique (React Admin), générateur de clients (Next.js, Vue, React, etc.), Mercure pour le temps réel.
5. **Extensibilité** : state providers, state processors, voters, normalizers pour brancher votre logique métier.
6. **Déploiement** : Docker et Helm/Kubernetes fournis dans la distribution officielle.

### Cas d'usage typiques

- **API métier** (B2B, B2C) : catalogue, commandes, utilisateurs.
- **Backend pour SPA / mobile** : une API unique pour web et apps.
- **Open Data / interopérabilité** : JSON-LD et vocabulaires (ex. Schema.org).
- **Prototypage** : valider une idée avec une API et un admin en quelques heures.

## Écosystème API Platform

### Cœur (Core)

- **Bibliothèque PHP** : crée des APIs hypermédia ou GraphQL.
- **Symfony** : framework sous-jacent (optionnel mais recommandé).
- **Doctrine ORM** : persistance par défaut dans la distribution (optionnelle).

### Distribution officielle

En téléchargeant la [distribution API Platform](https://github.com/api-platform/api-platform/releases), vous obtenez notamment :

- Un squelette d'API (Core + Symfony + Doctrine).
- Une **interface d'administration** (React Admin) générée dynamiquement à partir de la doc de l'API.
- Un **générateur de clients** pour Next.js, Nuxt, Vue, React, React Native, etc.
- **Mercure** pour les APIs temps réel / asynchrones.
- Des définitions **Docker** et des chartes **Helm** pour le déploiement.

### Formats et protocoles

| Format / protocole | Usage |
|--------------------|--------|
| **JSON-LD + Hydra** | Format par défaut, hypermédia, découvrabilité |
| **OpenAPI (Swagger)** | Documentation et UI interactive |
| **JSON:API** | Standard REST alternatif |
| **HAL** | Hypermédia léger |
| **GraphQL** | Requêtes flexibles (module séparé) |

## REST vs GraphQL avec API Platform

- **REST** : activé par défaut. Une ressource = une collection (ex. `/books`) et des items (ex. `/books/1`). Opérations GET, POST, PATCH, DELETE, etc.
- **GraphQL** : optionnel (package `api-platform/graphql`). Même modèle de ressources ; exposition en requêtes et mutations GraphQL, compatible Relay.

Vous pouvez proposer **les deux** sur le même projet : mêmes entités, deux interfaces (REST + GraphQL).

## Ce que cette formation couvrira

1. **Concepts fondamentaux** : ressource, opérations, sérialisation, state providers et processors.
2. **Fonctionnalités avancées** : filtres, tri, validation, sécurité, formats multiples.
3. **Exemples concrets** : API type bookshop (livres, avis) en PHP.
4. **Mise en pratique** : installation (Docker ou Symfony Flex), première ressource, bonnes pratiques et déploiement.

À la fin, vous saurez expliquer simplement ce qu'est API Platform, pourquoi l'utiliser, et comment structurer une petite API avec ressources, opérations et persistance.
