# Changelog

Tous les changements notables de ce projet seront documentés dans ce fichier.

Le format est basé sur [Keep a Changelog](https://keepachangelog.com/fr/1.0.0/),
et ce projet adhère au [Semantic Versioning](https://semver.org/lang/fr/).

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
