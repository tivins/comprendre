# Introduction aux stratégies de refactorisation de code legacy

## Qu’est-ce que le code legacy ?

Le **code legacy** n’est pas uniquement « du vieux code ». C’est du code difficile à faire évoluer : peu ou pas de tests, couplage fort, responsabilités mélangées, dépendances directes (BDD, APIs, fichiers). Souvent, personne ne maîtrise plus tout le comportement ; modifier une partie peut casser une autre sans qu’aucun test ne le signale.

En résumé : **legacy = code dont le changement est risqué parce que le comportement n’est pas documenté par des tests et que la structure complique les modifications localisées.**

## Pourquoi refactoriser ?

- **Réduire le risque** : chaque changement devient plus prévisible si des tests couvrent le comportement et si le code est découplé.
- **Permettre l’évolution** : ajouter une fonctionnalité ou corriger un bug sans tout casser.
- **Faciliter la compréhension** : responsabilités claires, dépendances explicites.

Refactoriser ne vise pas à « faire propre » pour le principe : c’est un moyen de rendre le code **modifiable en sécurité**. Sans objectif (nouvelle feature, bug récurrent, dette technique bloquante), le refactoring pur peut être du temps perdu — à discuter en équipe.

## Refactoring incrémental vs réécriture

**Réécriture complète** : remplacer tout le module ou l’application. Risques : longueur du chantier, divergence de comportement, double maintenance pendant la transition. Souvent tentant ; rarement maîtrisable sur du code critique.

**Refactoring incrémental** : avancer par petits pas, en s’appuyant sur des tests (de caractérisation d’abord) et sur des stratégies d’encapsulation (Strangler Fig, branche par abstraction). Le système reste livrable à tout moment ; le risque est réparti dans le temps.

Cette documentation privilégie l’**incrémental** : encapsuler le legacy, ajouter des tests, remplacer ou réécrire morceau par morceau.

## Risques et bénéfices

### Risques du refactoring legacy

- **Régression silencieuse** : pas de tests → un changement peut casser un comportement non documenté.
- **Sur-refactoring** : tout déplier alors qu’une petite modification suffisait.
- **Couplage caché** : des dépendances globales (singletons, variables statiques, BDD directe) rendent les changements en cascade.

### Bénéfices quand c’est bien mené

- **Confiance** : les tests de caractérisation documentent le comportement actuel ; les refactorings suivants s’appuient sur une base stable.
- **Découplage progressif** : en introduisant des points de couture (seams), on isole le legacy et on peut le remplacer par étapes.
- **Livraison continue** : pas de « big bang » ; chaque itération livre une version fonctionnelle.

## Ce que cette doc couvre

- **Concepts** : tests de caractérisation, seams (points de couture), encapsuler puis remplacer.
- **Stratégies** : Strangler Fig, branche par abstraction, exécution parallèle, feature flags.
- **Exemples** : extraction de service, ajout de tests sur du code non testé, remplacement d’une dépendance.
- **Pratique** : quand refactoriser, pièges courants, intégration dans le workflow.

## Ce que cette doc ne couvre pas

- **Réécriture complète** : choix d’une nouvelle stack, migration « big bang » — sujet distinct.
- **Refactoring sur code déjà bien testé** : les catalogues de refactorings (extract method, rename, etc.) sont bien documentés ailleurs (ex. Martin Fowler, *Refactoring*).
- **Outillage détaillé** : on présente les idées ; l’outil (IDE, framework de tests) dépend du langage et du projet.

## Prochaines étapes

La suite de la documentation détaille :

- **[Concepts fondamentaux](./02-concepts-fondamentaux.md)** : tests de caractérisation, seams, encapsuler puis remplacer
- **[Stratégies et approches](./03-strategies-et-approches.md)** : Strangler Fig, branche par abstraction, exécution parallèle, feature flags
- **[Exemples concrets](./04-exemples-concrets.md)** : extraction de service, tests sur code non testé, remplacement d’une dépendance
- **[Mise en pratique](./05-mise-en-pratique.md)** : quand refactoriser, pièges, intégration dans le workflow

---

**Rappel** : Le but du refactoring legacy n’est pas la perfection, mais la **réduction du risque** et la **capacité à faire évoluer** le code. Avancer par petits pas et documenter le comportement par des tests avant de modifier la structure.
