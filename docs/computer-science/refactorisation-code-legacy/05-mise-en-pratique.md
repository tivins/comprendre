# Mise en pratique

Ce guide aide à appliquer une approche cohérente du refactoring legacy : quand refactoriser, pièges courants et intégration dans le workflow.

## Quand refactoriser

### Refactoriser en priorité quand

- Une **zone est souvent modifiée** (bugs répétés, nouvelles features) et le coût de chaque changement est élevé (couplage, absence de tests).
- Une **dépendance critique** (SDK déprécié, API qui change) doit être remplacée ; l’encapsulation et la branche par abstraction réduisent le risque.
- Une **nouvelle fonctionnalité** exige un comportement proche de l’existant ; extraire un service ou introduire un seam évite de dupliquer du legacy.

### Différer ou limiter le refactoring quand

- La zone est **stable et rarement touchée** ; le coût du refactoring peut dépasser le bénéfice.
- Pas de **objectif clair** (ni feature, ni bug, ni dette bloquante) ; le refactoring pur peut être reporté ou ciblé sur un sous-ensemble.
- L’équipe n’a **pas la bande passante** pour maintenir des tests et une double implémentation pendant la transition ; mieux vaut des pas plus petits ou un périmètre réduit.

En résumé : **prioriser les zones à fort impact et à forte fréquence de changement** ; accepter de laisser du legacy « tranquille » ailleurs tant que le coût de le toucher reste supérieur au bénéfice.

---

## Pièges à éviter

### Refactoriser sans tests de caractérisation

Modifier du code non testé sans avoir au préalable documenté le comportement actuel par des tests de caractérisation multiplie le risque de régression. Même un seul test par scénario critique aide ; idéalement, on ajoute les tests avant de toucher à la structure.

### Tout réécrire d’un coup

Une réécriture complète d’un module ou d’une application prolonge la période où deux comportements coexistent (ancien vs nouveau) et complique les corrections. Préférer **encapsuler puis remplacer** par étapes, avec des bascules progressives (feature flag, pourcentage de trafic).

### Introduire des seams partout sans priorité

Chaque seam a un coût (modification des signatures, propagation des dépendances). Introduire des seams sur tout le legacy en une fois est coûteux et peut déstabiliser le système. Cibler les **dépendances qui bloquent les tests** ou le remplacement (accès BDD direct, appel API, singleton).

### Laisser proliférer les feature flags

Les flags non nettoyés compliquent le code et la compréhension. Nommer clairement, documenter la durée de vie prévue, et prévoir un processus de suppression (date de revue, ticket de nettoyage) une fois la bascule terminée.

### Ignorer la priorisation

La dette technique n’est pas à rembourser intégralement. Prioriser : quelles parties génèrent le plus de bugs ou bloquent le plus les évolutions ? Investir là ; accepter de laisser le reste en l’état pour l’instant.

---

## Intégration dans le workflow

### Local

- **Avant tout refactoring** sur une zone legacy : ajouter au moins un ou deux tests de caractérisation (comportement actuel).
- **Après introduction d’un seam** : exécuter les tests existants et les nouveaux ; vérifier que le comportement observable ne change pas.
- **Branche par abstraction** : faire valider l’interface (nommage, paramètres) en revue de code pour éviter des changements en cascade plus tard.

### CI / livraison

- Les **tests de caractérisation** doivent rester verts à chaque commit ; les traiter comme des tests de non-régression.
- En cas de **feature flag** pour basculer ancien/nouveau code : prévoir des tests (ou des jobs) qui exécutent les deux chemins tant que les deux sont supportés, puis supprimer l’ancien chemin et le flag une fois la migration terminée.
- Documenter brièvement la **stratégie en cours** (Strangler, branche par abstraction) dans le README ou les ADR pour que l’équipe sache où en est la migration.

### Revue de code

- Vérifier que les **refactorings** sont accompagnés de tests (caractérisation ou unitaires) quand le code était non testé.
- Vérifier que les **seams** (nouvelles interfaces, injection) ne dégradent pas inutilement la lisibilité ; éviter les abstractions précoces sur du code encore peu touché.

---

## Questions à se poser en équipe

- **Quelle zone legacy nous coûte le plus** (temps, bugs, peur de toucher) ? C’est un bon candidat pour un premier chantier de refactoring incrémental.
- **Avons-nous le temps** de maintenir des tests et éventuellement deux implémentations pendant la transition ? Si non, réduire le périmètre ou les pas.
- **Le comportement actuel est-il correct ?** Les tests de caractérisation figent ce comportement ; s’il est bugué, décider de le corriger (et mettre à jour le test) avant ou pendant le refactoring.
- **Dans notre contexte, le coût de la complexité** (Strangler, branche par abstraction, feature flags) **vaut-il la chandelle** par rapport à une réécriture ciblée ou à un statu quo ? La réponse dépend du projet et de la durée de vie attendue du legacy.

---

## Prochaines étapes

- Revenir aux **[concepts](./02-concepts-fondamentaux.md)** et **[stratégies](./03-strategies-et-approches.md)** pour choisir l’approche adaptée à votre périmètre.
- Consulter **[Working Effectively with Legacy Code](https://www.goodreads.com/book/show/44919.Working_Effectively_with_Legacy_Code)** (Michael Feathers) et **[Refactoring](https://refactoring.com/)** (Martin Fowler) pour approfondir.

---

**Rappel** : Refactoriser du legacy est un processus continu et priorisé, pas un one-shot. Petits pas, tests de caractérisation, seams ciblés et stratégies d’encapsulation permettent de progresser sans tout casser.
