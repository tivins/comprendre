# Mise en pratique avec API Platform

## 1. Choisir une méthode d'installation

Deux options courantes :

- **Distribution officielle (Docker)** : squelette complet (API + PWA + admin + Mercure), idéal pour découvrir l’écosystème.
- **Symfony Flex** : intégration dans un projet Symfony existant, plus léger.

## 2. Installation avec la distribution (Docker)

### Prérequis

- [Docker](https://docs.docker.com/get-docker/) et Docker Compose.
- Git (pour cloner le dépôt ou utiliser le template GitHub).

### Étapes

1. Télécharger la [distribution API Platform](https://github.com/api-platform/api-platform/releases) (préférer l’archive `.tar.gz` pour éviter des soucis de droits), ou créer un dépôt à partir du [template GitHub](https://github.com/api-platform/api-platform).

2. Aller dans le répertoire du projet et lancer les conteneurs :

```bash
docker compose build --no-cache
docker compose up --wait
```

3. Vérifier que les services tournent :
   - **API** : `https://localhost/docs/` (Swagger UI).
   - **PWA** (si incluse) : `https://localhost/`.
   - **Admin** : `https://localhost/admin/`.

Le navigateur peut demander d’accepter le certificat TLS auto-signé.

### Créer une ressource dans le conteneur PHP

Exécuter les commandes dans le conteneur `php` :

```bash
docker compose exec php bin/console make:entity --api-resource
```

Répondre aux questions (nom de l’entité, propriétés). L’entité est créée avec le mapping Doctrine et l’attribut `#[ApiResource]`.

Puis créer/mettre à jour la base et les migrations :

```bash
docker compose exec php bin/console doctrine:database:create
docker compose exec php bin/console doctrine:migrations:diff
docker compose exec php bin/console doctrine:migrations:migrate
```

Recharger `https://localhost/docs/` : la nouvelle ressource apparaît dans Swagger.

## 3. Installation avec Symfony Flex

### Prérequis

- PHP 8.2+ (recommandé), Composer, extension PHP habituelle (json, pdo, etc.).
- [Symfony CLI](https://symfony.com/download) (optionnel mais pratique).

### Étapes

1. Créer un projet Symfony :

```bash
symfony new bookshop-api
cd bookshop-api
```

2. Installer API Platform :

```bash
symfony composer require api
```

3. Créer la base et le schéma :

```bash
symfony console doctrine:database:create
symfony console doctrine:schema:create
```

4. Démarrer le serveur :

```bash
symfony serve
```

L’API est disponible à `http://localhost:8000/api/` (ou l’URL affichée par `symfony serve`). La documentation Swagger est généralement à `http://localhost:8000/api/` ou une route dédiée selon la config.

### Créer une première ressource

```bash
symfony console make:entity --api-resource
```

Compléter le questionnaire (nom, propriétés). Puis :

```bash
symfony console doctrine:migrations:diff
symfony console doctrine:migrations:migrate
```

Tester un `GET` sur la collection et un `POST` depuis Swagger ou un client HTTP.

## 4. Bonnes pratiques

### Modèle et ressources

- **Une ressource = un type métier cohérent** (Book, Review, Order, etc.). Éviter d’exposer des structures techniques brutes.
- **Validation** : mettre des contraintes `Assert\*` sur toutes les entrées (champs obligatoires, formats, plages). Les erreurs 422 structurées facilitent le travail du front.
- **Sérialisation** : utiliser des **groupes** pour distinguer liste / détail et lecture / écriture. Ne pas exposer par défaut des champs sensibles (mots de passe, tokens).

### Sécurité

- **Authentification** : JWT ou OAuth2 selon le besoin ; bien configurer les clés et les durées de vie.
- **Autorisation** : utiliser des **voters** et l’attribut `security` sur les opérations (et éventuellement sur les champs via les groupes et le contexte).
- **CORS** : configurer les origines autorisées en production.

### Performance

- **Pagination** : garder une taille de page raisonnable (ex. 30) et documenter les paramètres pour le client.
- **Filtres** : limiter les propriétés filtrables et les stratégies (exact, partial) pour éviter des requêtes trop lourdes.
- **Cache HTTP** : activer l’invalidation (Varnish ou équivalent) pour les ressources peu modifiées.
- **Doctrine** : vérifier les requêtes N+1 ; utiliser les jointures nécessaires (API Platform peut optimiser les JOIN selon la sérialisation).

### Maintenance

- **Versions d’API** : pour une évolution contrôlée, utiliser des préfixes d’URL (`/api/v1/`) ou des groupes de sérialisation par version.
- **Documentation** : compléter les descriptions des ressources et opérations (OpenAPI) pour faciliter l’intégration des clients.
- **Tests** : tester les opérations CRUD, la validation et les règles de sécurité (tests fonctionnels ou E2E).

## 5. Aller plus loin

- **State providers / processors personnalisés** : pour une logique métier complexe ou une persistance non Doctrine (NoSQL, API externe).
- **Opérations custom** : actions spécifiques (ex. `POST /books/{id}/publish`) avec des processeurs dédiés.
- **GraphQL** : `composer require api-platform/graphql` puis exposition des mêmes ressources en GraphQL.
- **Admin** : l’admin React (distribution) se génère à partir de la doc Hydra ; personnalisation via les métadonnées et les groupes.
- **Déploiement** : utiliser les chartes Helm/Kubernetes de la distribution ou déployer l’API comme une application PHP classique (Symfony) derrière Nginx/Apache et PHP-FPM.

## 6. Résumé des commandes utiles

| Action | Distribution (Docker) | Symfony Flex |
|--------|------------------------|--------------|
| Démarrer | `docker compose up --wait` | `symfony serve` |
| Créer une entité API | `docker compose exec php bin/console make:entity --api-resource` | `symfony console make:entity --api-resource` |
| Migrations | `docker compose exec php bin/console doctrine:migrations:diff` puis `migrate` | `symfony console doctrine:migrations:diff` puis `migrate` |
| Installer GraphQL | `docker compose exec php composer require api-platform/graphql` | `symfony composer require api-platform/graphql` |

En suivant ce guide, vous pouvez installer API Platform, exposer vos premières ressources, les valider et les sécuriser, puis étendre l’API avec filtres, GraphQL et déploiement selon vos besoins.
