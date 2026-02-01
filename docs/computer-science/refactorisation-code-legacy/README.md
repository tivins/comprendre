# Stratégies de refactorisation de code legacy

## Introduction

Le **code legacy** désigne du code existant, souvent critique et peu documenté, dont la structure complique les évolutions et les corrections. Le refactoriser sans tout casser demande une approche progressive : tests de caractérisation, stratégies d’encapsulation (Strangler Fig, branche par abstraction), et découpage par petits pas. Cette documentation présente les concepts et stratégies courantes pour aborder le legacy de façon pragmatique, sans réécriture hasardeuse.

Elle s’adresse à des développeurs à l’aise avec un langage applicatif (Java, PHP, TypeScript, etc.), qui doivent faire évoluer ou sécuriser du code ancien sans disposer d’une couverture de tests ni d’une architecture idéale.

## Objectifs de cette documentation

Cette documentation vous permettra de :

- Comprendre ce qu’on entend par code legacy et pourquoi le refactoriser de façon incrémentale
- Maîtriser les concepts clés : tests de caractérisation, points de couture (seams), encapsuler puis remplacer
- Connaître les stratégies courantes : Strangler Fig, branche par abstraction, exécution parallèle, feature flags
- Voir des exemples concrets : extraction de service, ajout de tests sur du code non testé, remplacement d’une dépendance
- Mettre en pratique une approche cohérente : quand refactoriser, pièges à éviter, intégration dans le workflow

## Structure de la documentation

1. **[Introduction](./01-introduction.md)** — Enjeux du code legacy, refactoring incrémental, risques et bénéfices
2. **[Concepts fondamentaux](./02-concepts-fondamentaux.md)** — Tests de caractérisation, seams, encapsuler puis remplacer
3. **[Stratégies et approches](./03-strategies-et-approches.md)** — Strangler Fig, branche par abstraction, exécution parallèle, feature flags
4. **[Exemples concrets](./04-exemples-concrets.md)** — Extraction de service, tests sur code non testé, remplacement d’une dépendance
5. **[Mise en pratique](./05-mise-en-pratique.md)** — Quand refactoriser, pièges courants, intégration dans le workflow

## Pour qui ?

- Développeurs qui maintiennent ou font évoluer du code ancien sans tests ou mal structuré
- Équipes qui hésitent entre « tout réécrire » et « ne rien toucher »
- Toute personne souhaitant structurer une approche progressive du refactoring legacy

## Prérequis

- Pratique du développement (Java, PHP, TypeScript ou équivalent)
- Notions de tests unitaires et d’injection de dépendances (voir [Types de tests](../types-de-tests/README.md), [Dependency Injection](../dependency-injection/README.md))
- Idéalement : notions de design patterns et de couplage (voir [SOLID](../solid/README.md))

## Langage utilisé dans les exemples

Les exemples sont en **pseudo-code inspiré de Java/PHP/TypeScript** : interfaces, classes, injection au constructeur, volontairement lisibles pour plusieurs écosystèmes.

---

**Note** : Refactoriser du legacy ne signifie pas tout réécrire d’un coup. Les stratégies décrites visent à réduire le risque en avançant par petits pas, en s’appuyant sur des tests de caractérisation et des points de couture pour isoler puis remplacer le code problématique.

---

Liens :

* [Michael Feathers — Working Effectively with Legacy Code](https://www.goodreads.com/book/show/44919.Working_Effectively_with_Legacy_Code)
* [Martin Fowler — Strangler Fig Application](https://martinfowler.com/bliki/StranglerFigApplication.html)
* [Martin Fowler — Refactoring](https://refactoring.com/)
