# Design Pattern Decorator

## Introduction

Le **Decorator** (ou Décorateur) est un pattern structurel qui permet d'attacher dynamiquement des responsabilités supplémentaires à un objet. Il fournit une alternative flexible à l'héritage pour étendre les fonctionnalités d'une classe sans en modifier le code source.

Ce pattern fait partie des 23 design patterns du "Gang of Four" (GoF) et s'inscrit dans la catégorie des patterns structurels.

## Objectifs de cette documentation

Cette documentation vous permettra de :
- Comprendre le problème que le Decorator résout et son intention
- Maîtriser la structure et les rôles des composants (Composant, Decorator concret)
- Voir des exemples concrets : café, logging, validation, I/O
- Savoir quand utiliser le Decorator et quand lui préférer d'autres solutions
- Mettre en pratique le pattern dans vos projets

## Structure de la documentation

1. **[Introduction au pattern Decorator](./01-introduction.md)** - Problème, solution, analogie
2. **[Structure et principe](./02-structure-et-principe.md)** - Composants, diagramme, principe Open/Closed
3. **[Exemples concrets](./03-exemples-concrets.md)** - Café, logger, validation, flux de données
4. **[Mise en pratique](./04-mise-en-pratique.md)** - Quand l'utiliser, alternatives, pièges à éviter

---

**Note** : Le Decorator est un outil pour étendre le comportement de façon modulaire. Il ne doit pas être utilisé pour tout étendre : réserver son usage aux cas où les combinaisons de fonctionnalités sont nombreuses ou évolutives.

---

Liens :

* [Refactoring Guru - Decorator][1]
* [Design Patterns (GoF)][2]

[1]: https://refactoring.guru/design-patterns/decorator
[2]: https://en.wikipedia.org/wiki/Design_Patterns
