# API Platform

## Introduction

**API Platform** est un framework full stack dédié aux projets pilotés par API. Construit sur Symfony et créé par Kévin Dunglas, il permet de créer des APIs web hypermédia (REST) et GraphQL conformes aux standards (JSON-LD/Hydra, OpenAPI, JSON:API, HAL) en exposant simplement des classes PHP comme ressources.

## Objectifs de cette documentation

Cette documentation vous permettra de :
- Comprendre ce qu'est API Platform et dans quels cas l'utiliser
- Maîtriser les concepts fondamentaux : ressources, opérations, sérialisation
- Découvrir les fonctionnalités avancées : filtres, validation, sécurité, GraphQL
- Suivre des exemples concrets (API type bookshop)
- Mettre en pratique : installation, première API, bonnes pratiques

## Structure de la documentation

1. **[Introduction](./01-introduction.md)** - Qu'est-ce qu'API Platform, écosystème, avantages
2. **[Concepts fondamentaux](./02-concepts-fondamentaux.md)** - Ressources, opérations, sérialisation, persistance
3. **[Fonctionnalités avancées](./03-fonctionnalites-avancees.md)** - Filtres, validation, sécurité, GraphQL, formats
4. **[Exemples concrets](./04-exemples-concrets.md)** - Cas d'usage avec code PHP
5. **[Mise en pratique](./05-mise-en-pratique.md)** - Installation, première API, déploiement

## Stack typique

- **Backend** : API Platform Core, Symfony, Doctrine ORM (optionnel)
- **Formats** : JSON-LD (défaut), JSON:API, HAL, OpenAPI/Swagger
- **Frontend** : générateur de clients (Next.js, Nuxt, Vue, React, React Native)
- **Admin** : interface dynamique basée sur React Admin

---

**Note** : API Platform est idéal pour du prototypage et du RAD, mais il est conçu pour des projets API complexes, au-delà du simple CRUD, grâce à ses points d'extension (state providers, processors, voters).

---

Liens :

* [Documentation officielle API Platform][1]
* [Symfony][2]

[1]: https://api-platform.com/docs/
[2]: https://symfony.com/
