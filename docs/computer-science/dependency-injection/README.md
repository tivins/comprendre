# Injection de dépendances

## Introduction

L'**injection de dépendances** (Dependency Injection, DI) est une technique de conception logicielle qui consiste à fournir à un objet les dépendances dont il a besoin, plutôt que de les créer ou les récupérer lui-même. Elle permet de découpler les composants, d'améliorer la testabilité et de respecter le principe d'inversion des dépendances (DIP) des principes SOLID.

Cette pratique est au cœur des frameworks modernes (Spring, Jakarta EE, Angular, etc.) et constitue une compétence essentielle pour tout développeur orienté objet.

## Objectifs de cette documentation

Cette documentation vous permettra de :
- Comprendre ce qu'est l'injection de dépendances et pourquoi elle est utile
- Distinguer les différents types d'injection (constructeur, setter, interface)
- Savoir quand et comment appliquer l'injection de dépendances
- Découvrir le rôle des conteneurs IoC (Inversion of Control)
- Voir des exemples concrets en pseudo-code Java
- Mettre en pratique la DI dans vos projets

## Structure de la documentation

1. **[Introduction à l'injection de dépendances](./01-introduction.md)** - Définition, problème sans DI, solution, lien avec le DIP
2. **[Types et mécanismes](./02-types-et-mecanismes.md)** - Injection par constructeur, par setter, par interface ; conteneurs IoC
3. **[Exemples concrets](./03-exemples-concrets.md)** - Service métier, repository, notification, tests unitaires
4. **[Mise en pratique](./04-mise-en-pratique.md)** - Quand l'utiliser, pièges à éviter, intégration dans un projet

## Pour qui ?

Cette documentation s'adresse à :
- Développeurs souhaitant réduire le couplage et améliorer la testabilité
- Étudiants en informatique découvrant les bonnes pratiques de conception
- Équipes adoptant des frameworks basés sur l'IoC (Spring, etc.)
- Toute personne souhaitant approfondir le principe DIP (SOLID)

## Prérequis

- Connaissance de base en programmation orientée objet
- Notions de classe, interface, constructeur et polymorphisme
- Idéalement : avoir lu le [principe d'inversion des dépendances (DIP)](../solid/06-dependency-inversion.md)

## Langage utilisé dans les exemples

Les exemples sont rédigés en **pseudo-code style Java** (interfaces, classes, typage explicite) pour rester proches des concepts OOP classiques et du style des autres documentations du projet.

---

**Note** : L'injection de dépendances n'est pas un objectif en soi mais un moyen d'obtenir un code découplé, testable et maintenable. Elle doit être appliquée avec discernement.

---

Liens :

* [Martin Fowler - Inversion of Control Containers and the Dependency Injection pattern][1]
* [Refactoring Guru - Dependency Injection][2]

[1]: https://martinfowler.com/articles/injection.html
[2]: https://refactoring.guru/dependency-injection
