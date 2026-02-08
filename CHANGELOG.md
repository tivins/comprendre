# Changelog

Tous les changements notables de ce projet seront documentés dans ce fichier.

Le format est basé sur [Keep a Changelog](https://keepachangelog.com/fr/1.0.0/),
et ce projet adhère au [Semantic Versioning](https://semver.org/lang/fr/).

## [1.8.7] - 2026-02-07

### Modifié
- Renommage de la doc « Stratégie de tests pour un moteur de recherche complexe (Drupal 7) » : dossier `strategie-tests-moteur-recherche` → `cases/tester-un-ensemble`, titre → « tester les ensembles »
- Lien mis à jour dans le sommaire : [tester les ensembles](docs/computer-science/cases/tester-un-ensemble/README.md)

## [1.8.6] - 2026-02-06

### Ajouté
- Documentation sur la stratégie de tests pour un moteur de recherche complexe Drupal 7 (computer-science)
  - Introduction (contexte Drupal 7, nature du problème, diagnostic de l'approche actuelle PHPUnit + données pré-insérées)
  - Concepts fondamentaux (couches testables d'un moteur de recherche, niveaux de test, découplage dans un contexte legacy)
  - Stratégies et approches (tests unitaires des filtres et du traitement, intégration avec BDD réelle, fonctionnel, tables temporaires, hooks)
  - Exemples concrets (PHP/PHPUnit : résolution de filtres, scoring, recherche en base, table temporaire, introspection de requête)
  - Mise en pratique (évaluation de l'existant, plan d'amélioration en 5 étapes, intégration CI, pièges à éviter)
- Lien vers [Stratégie de tests moteur de recherche](docs/computer-science/cases/tester-un-ensemble/README.md) dans le sommaire de `docs/README.md` (dossier renommé en 1.8.7)

## [1.8.5] - 2026-02-01

### Ajouté
- Documentation sur les stratégies de refactorisation de code legacy (computer-science)
  - Introduction (enjeux du legacy, refactoring incrémental vs réécriture, risques et bénéfices)
  - Concepts fondamentaux (tests de caractérisation, seams, encapsuler puis remplacer)
  - Stratégies et approches (Strangler Fig, branche par abstraction, exécution parallèle, feature flags)
  - Exemples concrets (introduction de seams, tests de caractérisation, remplacement d’un service de paiement)
  - Mise en pratique (quand refactoriser, pièges à éviter, intégration dans le workflow)
- Lien vers [Refactorisation code legacy](docs/computer-science/refactorisation-code-legacy/README.md) dans le sommaire de `docs/README.md`

## [1.8.4] - 2026-02-01

### Ajouté
- Documentation sur les tests des requêtes MySQL complexes et du traitement des données (computer-science)
  - Introduction (enjeux, ce qu’on teste, vocabulaire)
  - Concepts fondamentaux (découplage requête / traitement, types de tests, reproductibilité)
  - Stratégies et outils (Testcontainers, fixtures, mocks du repository, assertions)
  - Exemples concrets (rapport avec agrégation, requête multi-tables, traitement après fetch)
  - Mise en pratique (quand quoi tester, pièges, intégration dans le workflow et la CI)
- Lien vers [Tests requêtes MySQL complexes](docs/computer-science/tests-requetes-mysql-complexes/README.md) dans le sommaire de `docs/README.md`

## [1.8.3] - 2026-01-31

### Ajouté
- Documentation sur les types de tests (computer-science)
  - Introduction (pourquoi distinguer les types, vocabulaire de base)
  - Types et pyramide (unitaire, intégration, E2E, contractuels ; pyramide des tests)
  - Exemples concrets (logique métier, service + repository, API HTTP, E2E)
  - Mise en pratique (quand écrire quoi, pièges à éviter, intégration dans le workflow)
- Lien vers [Types de tests](docs/computer-science/types-de-tests/README.md) dans le sommaire de `docs/README.md`

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
