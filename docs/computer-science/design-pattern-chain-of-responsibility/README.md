# Design Pattern Chaîne de responsabilité

## Introduction

La **Chaîne de responsabilité** (Chain of Responsibility) est un pattern comportemental qui évite de coupler l'émetteur d'une requête à ses récepteurs en donnant à plusieurs objets la possibilité de traiter la requête. Les objets sont chaînés et la requête circule le long de la chaîne jusqu'à ce qu'un objet la prenne en charge.

Ce pattern fait partie des 23 design patterns du "Gang of Four" (GoF) et s'inscrit dans la catégorie des patterns comportementaux.

## Objectifs de cette documentation

Cette documentation vous permettra de :
- Comprendre le problème que la Chaîne de responsabilité résout et son intention
- Maîtriser la structure et les rôles des composants (Handler abstrait, handlers concrets, chaînage)
- Voir des exemples concrets : validation, gestion d'erreurs HTTP, pipeline de traitement
- Savoir quand utiliser la Chaîne de responsabilité et quand lui préférer d'autres solutions
- Mettre en pratique le pattern dans vos projets

## Structure de la documentation

1. **[Introduction au pattern Chaîne de responsabilité](./01-introduction.md)** - Problème, solution, analogie
2. **[Structure et principe](./02-structure-et-principe.md)** - Handler, successeur, passage de la requête
3. **[Exemples concrets](./03-exemples-concrets.md)** - Validation, erreurs HTTP, pipeline, niveaux de support
4. **[Mise en pratique](./04-mise-en-pratique.md)** - Quand l'utiliser, alternatives, pièges à éviter

## Pour qui ?

Cette documentation s'adresse à :
- Développeurs souhaitant découpler l'émetteur d'une requête de ses traitants possibles
- Architectes cherchant une solution flexible pour des pipelines de traitement
- Étudiants en informatique découvrant les patterns comportementaux
- Équipes voulant éviter les longues séries de `if/else` ou de `switch` pour router des requêtes

## Prérequis

- Connaissance de base en programmation orientée objet
- Notions de classe, interface, héritage et polymorphisme
- Idéalement : avoir lu l'[introduction aux Design Patterns](../design-pattern/01-introduction.md)

## Langage utilisé dans les exemples

Les exemples sont rédigés en **pseudo-code style Java** (interfaces, classes, `implements`/`extends`, typage explicite) pour rester proches des concepts OOP classiques et du style des autres documentations du projet.

---

**Note** : La Chaîne de responsabilité est un outil pour découpler l'émetteur des traitants et pour faire circuler une requête jusqu'à ce qu'elle soit prise en charge. Elle ne doit pas être utilisée lorsque un seul objet doit toujours traiter la requête (un simple appel de méthode suffit).

---

Liens :

* [Refactoring Guru - Chain of Responsibility][1]
* [Design Patterns (GoF)][2]

[1]: https://refactoring.guru/design-patterns/chain-of-responsibility
[2]: https://en.wikipedia.org/wiki/Design_Patterns
