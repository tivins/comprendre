# Changelog

Tous les changements notables de ce projet seront documentés dans ce fichier.

Le format est basé sur [Keep a Changelog](https://keepachangelog.com/fr/1.0.0/),
et ce projet adhère au [Semantic Versioning](https://semver.org/lang/fr/).

## [1.8.2] - 2026-01-31

### Ajouté
- Documentation sur le spin (physique)
  - Introduction (qu’est-ce que le spin, place en physique quantique, importance)
  - Concepts fondamentaux (quantification, fermions et bosons, orientation haut/bas)
  - Propriétés et mesure (moment magnétique, Stern–Gerlach, résonance magnétique)
  - Exemples concrets (IRM, RMN, spintronique)
  - Mise en pratique (reformuler, lire une actualité, ressources, bonnes pratiques)
- Lien vers [Spin](docs/physique/spin/README.md) dans le sommaire de `docs/README.md`

## [1.8.1] - 2026-01-31

### Modifié
- Réorganisation des dossiers : un dossier par science au même niveau sous `docs/`
  - `docs/physique/noyau-atome/` (déplacé depuis `docs/noyau-atome/`) pour la physique
  - `docs/computer-science/` inchangé pour l’informatique
  - Lien mis à jour dans `docs/README.md`

## [1.8.0] - 2026-01-31

### Ajouté
- Documentation sur le noyau des atomes (physique, hors computer-science)
  - Introduction (qu’est-ce que le noyau, place dans l’atome, importance)
  - Concepts fondamentaux (protons, neutrons, numéro atomique Z, nombre de masse A, isotopes)
  - Structure et stabilité (force forte, répulsion électrique, radioactivité α/β/γ, demi-vie, fusion/fission)
  - Exemples concrets (énergie nucléaire, datation carbone 14, médecine imagerie/radiothérapie)
  - Mise en pratique (reformuler, lire une actualité, ressources CEA/IRSN, bonnes pratiques)
- Section « Physique » dans le sommaire de `docs/README.md` avec lien vers le noyau des atomes

## [1.7.0] - 2026-01-31

### Ajouté
- Lexique conception et architecture (`docs/lexique.md`) : couplage, découplage, responsabilité, dépendance, cohésion, interface/contrat — définitions courtes et liens vers les sujets concernés.
- Documentation sur le bus applicatif (Message Bus)
  - Introduction (définition, analogie avec le bus matériel, rôle dans l’architecture applicative)
  - Concepts fondamentaux (messages, handlers, middleware, enveloppe, routage, synchrone/asynchrone)
  - Types et usage (command bus, event bus, CQRS, intégration DDD)
  - Exemples concrets en PHP avec DDD (Symfony Messenger, agrégats, commandes et événements)
  - Mise en pratique (quand introduire un bus, choix d’implémentation, bonnes pratiques, pièges à éviter)

## [1.6.0] - 2026-01-31

### Ajouté
- Documentation sur l’identification, l’authentification et l’autorisation (IAA)
  - Introduction (définitions, pourquoi distinguer les trois notions, analogie)
  - Concepts fondamentaux (identification, authentification, autorisation en détail)
  - Exemples concrets (flux de connexion, API JWT, RBAC)
  - Mise en pratique (bonnes pratiques, pièges à éviter, intégration)

## [1.5.0] - 2026-01-31

### Ajouté
- Documentation complète sur le bus (système de transmission de données)
  - Introduction (définition, analogie, rôle dans l’architecture, maître/esclave)
  - Concepts fondamentaux (lignes données/adresses/contrôle, largeur, bande passante, arbitrage, synchrone/asynchrone)
  - Types et architectures (bus système, bus E/S, parallèle vs série, architecture multi-bus)
  - Exemples concrets (PCI/PCIe, USB, I2C, SPI, CAN, Ethernet)
  - Mise en pratique (critères de choix, bonnes pratiques, exemple capteur embarqué)

## [1.4.0] - 2026-01-30

### Ajouté
- Documentation complète sur le design pattern Chaîne de responsabilité (Chain of Responsibility)
  - Introduction (problème, solution, analogie, lien SOLID)
  - Structure et principe (Handler, successeur, passage de la requête)
  - Exemples concrets (validation, erreurs HTTP, niveaux d'approbation, pipeline)
  - Mise en pratique (quand l'utiliser, alternatives, pièges à éviter)

## [1.3.0] - 2026-01-30

### Ajouté
- Documentation complète sur l'injection de dépendances (Dependency Injection)
  - Introduction (définition, problème sans DI, solution, lien avec le DIP)
  - Types et mécanismes (injection par constructeur, setter, interface ; conteneurs IoC)
  - Exemples concrets en pseudo-code Java (service métier, repository, notification, tests)
  - Mise en pratique (quand l'utiliser, pièges à éviter, intégration SOLID et frameworks)

## [1.2.0] - 2026-01-30

### Ajouté
- Documentation complète sur le design pattern Decorator
  - Introduction au pattern Decorator (problème, solution, analogie, lien SOLID)
  - Structure et principe (composant, decorator, délégation, Open/Closed)
  - Exemples concrets (café, logger, validation, flux, HTTP)
  - Mise en pratique (quand l'utiliser, alternatives, pièges à éviter)

## [1.1.0] - 2026-01-30

### Ajouté
- Documentation complète sur API Platform
  - Introduction à API Platform (écosystème, REST/GraphQL)
  - Concepts fondamentaux (ressources, opérations, sérialisation, state providers/processors)
  - Fonctionnalités avancées (filtres, validation, sécurité, GraphQL, formats, Mercure)
  - Exemples concrets (API bookshop avec Book et Review)
  - Mise en pratique (installation Docker et Symfony Flex, bonnes pratiques)

## [1.0.0] - 2026-01-28

### Ajouté
- Documentation complète sur les Architecture Decision Records (ADR)
  - Introduction aux ADR
  - Structure et format des ADR
  - Exemples concrets
  - Guide de mise en pratique
- Documentation complète sur les Design Patterns
  - Introduction aux design patterns
  - Patterns créationnels
  - Patterns structurels
  - Patterns comportementaux
  - Exemples concrets
  - Guide de mise en pratique
- Documentation complète sur le Domain-Driven Design (DDD)
  - Concepts fondamentaux du DDD
  - Patterns tactiques
  - Patterns stratégiques
  - Exemples concrets
  - Guide de mise en pratique
- Documentation complète sur les principes SOLID
  - Introduction aux principes SOLID
  - Single Responsibility Principle (SRP)
  - Open/Closed Principle (OCP)
  - Liskov Substitution Principle (LSP)
  - Interface Segregation Principle (ISP)
  - Dependency Inversion Principle (DIP)
  - Exemples concrets
  - Guide de mise en pratique
- Guide pratique d'organisation du code (coding-helper.md)
  - Questions à se poser lors de la création de classes, méthodes et dossiers
  - Astuces mnémotechniques pour SOLID et DDD
  - Checklist rapide avant de créer du code
  - Guide de réflexion quotidienne pour le développement
