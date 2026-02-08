# Mise en pratique

Ce guide aide à **évaluer l'approche actuelle** (PHPUnit + données pré-insérées), à tracer un **plan d'amélioration progressif**, et à intégrer la stratégie de tests dans le workflow quotidien et la CI.

## 1. Évaluation de l'approche actuelle

### Ce qu'on a aujourd'hui

Des tests PHPUnit qui :

1. Insèrent des données en base (nœuds, taxonomie, champs) en amont.
2. Appellent le moteur de recherche avec des paramètres.
3. Vérifient les résultats (IDs, nombre, ordre).

### Verdict : viable, mais perfectible

**C'est déjà bien.** Beaucoup de projets Drupal 7 n'ont aucun test automatisé sur leur moteur de recherche. Avoir une suite PHPUnit qui valide les résultats est un filet de sécurité réel.

**Mais** cette suite est en réalité un ensemble de **tests d'intégration** exécutés via PHPUnit. Ce n'est pas un problème en soi — PHPUnit est un runner, pas un label « unitaire seulement ». Le problème est l'**absence de niveaux complémentaires** :

| Aspect                  | Situation actuelle                             | Risque                                           |
|-------------------------|------------------------------------------------|--------------------------------------------------|
| Niveau de test          | Tout en intégration (BDD réelle)               | Lenteur, fragilité, diagnostic difficile         |
| Isolation               | Variables selon l'implémentation               | Dépendance à l'ordre, données résiduelles        |
| Couverture de la logique | Implicite (testée via le pipeline complet)    | Régressions logiques masquées par les données    |
| Maintenance             | Fixtures à maintenir avec le schéma            | Coût croissant à chaque évolution                |
| Diagnostic d'échec      | « Le résultat est faux » — où est le bug ?     | Requête ? Filtres ? Traitement ?                 |

### Ce qui manque

- **Des tests unitaires** sur la logique de filtrage et de traitement, pour un feedback rapide et un diagnostic précis.
- **Une isolation claire** entre tests (transaction/rollback).
- **Une séparation des responsabilités** dans le code, qui permettrait de tester chaque couche indépendamment.

---

## 2. Plan d'amélioration progressif

L'erreur serait de tout refactoriser d'un coup. Voici un plan en étapes, chaque étape apportant de la valeur immédiate.

### Étape 1 : Stabiliser les tests d'intégration existants

**Objectif** : rendre les tests actuels fiables et reproductibles.

**Actions** :

- **Isolation par transaction** : chaque test (ou classe de tests) s'exécute dans une transaction qui est annulée en `tearDown`. Cela garantit que chaque test part du même état.
- **Fixtures explicites** : chaque classe de tests déclare les données dont elle a besoin. Pas de dépendance à « ce qui est en base ».
- **Assertions sur l'essentiel** : vérifier les IDs et le nombre de résultats, pas des détails susceptibles de changer (timestamps, IDs auto-générés non maîtrisés).

**Effort** : quelques heures à quelques jours selon la taille de la suite existante. Retour immédiat : moins de tests qui cassent pour de mauvaises raisons.

### Étape 2 : Extraire et tester la logique de filtrage

**Objectif** : tester en unitaire les règles de combinaison de filtres.

**Actions** :

- Identifier la logique de filtrage dans le code du moteur (« si ce paramètre est défini, on ajoute ce filtre ; si celui-ci est absent, on applique le défaut »).
- L'extraire dans une classe `SearchFilterResolver` (ou équivalent) avec une signature claire : `resolve(array $params): SearchFilters`.
- Écrire des tests unitaires pour chaque règle (filtre actif, inactif, combinaison, dépendance, défaut).

**Effort** : modéré. L'extraction est souvent un simple « copier la logique conditionnelle dans une nouvelle classe ». Les tests unitaires sont rapides à écrire une fois la classe en place.

**Retour** : feedback en millisecondes sur les règles de filtrage. Les bugs de type « tel filtre ne marche plus avec tel autre » sont détectés sans base de données.

### Étape 3 : Extraire et tester le traitement des résultats

**Objectif** : tester en unitaire la logique appliquée sur les résultats bruts.

**Actions** :

- Identifier le traitement post-requête (scoring, tri, dédoublonnage, pagination, exclusions).
- L'extraire dans une classe `SearchResultProcessor`.
- Écrire des tests unitaires avec des tableaux de données en entrée.

**Effort** : similaire à l'étape 2. Parfois plus simple car le traitement est déjà dans une fonction séparée.

**Retour** : les régressions sur le scoring ou le tri sont détectées sans exécuter de requête.

### Étape 4 : Ajouter des tests d'intégration ciblés

**Objectif** : compléter la couverture des requêtes et des tables temporaires.

**Actions** :

- Pour chaque filtre ou combinaison complexe : un test d'intégration dédié avec des fixtures minimales.
- Pour les tables temporaires : des tests qui vérifient le contenu intermédiaire (dans la même connexion).
- Regrouper les tests d'intégration pour partager les fixtures quand les scénarios sont proches (même classe de tests).

**Effort** : progressif. Commencer par les filtres les plus critiques ou les plus sujets aux bugs.

### Étape 5 : Quelques tests fonctionnels (smoke tests)

**Objectif** : valider le pipeline complet sur les scénarios les plus importants.

**Actions** :

- Trois à cinq tests fonctionnels qui exécutent le moteur « comme en vrai » (avec bootstrap Drupal si nécessaire).
- Couvrir : recherche par défaut, filtre majeur, cas vide, cas d'erreur.

**Effort** : dépend du bootstrap Drupal. Si les tests d'intégration existants font déjà ce travail, l'apport marginal est faible.

---

## 3. Structure finale de la suite de tests

Après les cinq étapes, la suite ressemble à ceci :

```
tests/
├── Unit/
│   ├── SearchFilterResolverTest.php    ← Logique de filtrage (rapide)
│   ├── SearchResultProcessorTest.php   ← Traitement des résultats (rapide)
│   └── ...
├── Integration/
│   ├── SearchEngineIntegrationTest.php ← Requêtes + BDD (fixtures, rollback)
│   ├── SearchTempTableTest.php         ← Tables temporaires
│   └── ...
└── Functional/
    └── SearchModuleSmokeTest.php       ← Pipeline complet (Drupal bootstrap)
```

Configuration PHPUnit (exemple) :

```xml
<phpunit>
    <testsuites>
        <testsuite name="unit">
            <directory>tests/Unit</directory>
        </testsuite>
        <testsuite name="integration">
            <directory>tests/Integration</directory>
        </testsuite>
        <testsuite name="functional">
            <directory>tests/Functional</directory>
        </testsuite>
    </testsuites>
</phpunit>
```

On peut exécuter les tests unitaires seuls (`phpunit --testsuite unit`) pour un feedback rapide, et la suite complète avant un push ou en CI.

---

## 4. Intégration dans le workflow

### Au quotidien (développement local)

- **Tests unitaires** : à chaque modification de la logique de filtrage ou du traitement. Exécution en secondes.
- **Tests d'intégration** : avant un commit ou un push, ou quand on modifie une requête / un filtre qui touche à la base.
- **Tests fonctionnels** : avant une release ou un déploiement, ou quand on modifie le module dans son ensemble.

### En CI (pipeline)

| Étape CI          | Tests exécutés          | Condition de blocage |
|-------------------|-------------------------|----------------------|
| Chaque push / PR  | Unitaires               | Échec = PR bloquée   |
| Chaque push / PR  | Intégration (MySQL)     | Échec = PR bloquée   |
| Avant release     | Fonctionnels (Drupal)   | Échec = release bloquée |

Pour les tests d'intégration en CI, une instance MySQL est nécessaire. Options :

- **Service MySQL dans le pipeline** (GitHub Actions `services`, GitLab CI `services`, Jenkins plugin).
- **Testcontainers** (si Docker est disponible dans le runner CI).
- **Base MySQL dédiée au CI** (moins isolée mais plus simple).

---

## 5. Pièges à éviter

### « On refactorise tout avant de tester »

Non. Commencer par stabiliser les tests existants (étape 1), puis extraire progressivement. Chaque extraction est couverte par les tests d'intégration existants — qui servent de filet de sécurité pendant le refactoring.

### Fixtures qui dérivent

Les fixtures doivent évoluer avec le schéma. Si un champ est ajouté en base, les tests d'intégration qui insèrent des données doivent être mis à jour. Centraliser les fixtures (helpers `insertNode`, `insertTaxonomyIndex`) pour minimiser les endroits à modifier.

### Tests d'intégration qui testent la logique

Si un test d'intégration échoue et que le problème est dans la logique de filtrage (pas dans la requête), c'est un signal : cette logique devrait être testée en unitaire. Les tests d'intégration devraient principalement valider le SQL, pas les règles métier.

### Ignorer l'ordre des résultats

Si le moteur doit retourner les résultats dans un ordre précis (score, date), tester cet ordre explicitement. Si l'ordre n'est pas garanti (pas de `ORDER BY`), trier avant de comparer, ou utiliser des assertions d'ensemble (`assertContains`, `assertCount`).

### Tests qui dépendent de l'environnement

Un test qui passe en local mais échoue en CI (ou inversement) signale un problème d'isolation : données résiduelles, connexion différente, version de MySQL différente. Viser la reproductibilité : fixtures explicites, base de test isolée, même version MySQL en local et en CI.

---

## 6. Questions à se poser

- **Quelles parties du moteur causent le plus de bugs ?** Commencer par extraire et tester celles-là en priorité.
- **Les tests existants sont-ils reproductibles ?** Si non, l'étape 1 (isolation par transaction) est prioritaire.
- **Le coût d'extraction est-il justifié ?** Pour une logique de filtrage complexe (10+ règles, dépendances entre filtres), oui sans hésiter. Pour un filtre simple (un `WHERE` sur un champ), un test d'intégration suffit.
- **L'équipe est-elle à l'aise avec les mocks et l'injection ?** Si c'est nouveau (adoption SOLID récente), commencer par des classes simples avec des tests sans mock (entrée tableau → sortie tableau) avant d'introduire des doubles de test.

Le bon dosage dépend du contexte : taille de l'équipe, fréquence des changements sur le moteur, criticité de la recherche pour les utilisateurs. Mieux vaut une suite de tests modeste mais maintenue qu'une usine à gaz qu'on finit par ignorer.

---

## Ressources

- [PHPUnit — Documentation](https://docs.phpunit.de/)
- [Martin Fowler — Practical Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html)
- [Michael Feathers — Working Effectively with Legacy Code](https://www.oreilly.com/library/view/working-effectively-with/0131177052/)
- [Types de tests](../../types-de-tests/README.md) — rappel des niveaux (unitaire, intégration, E2E)
- [Tests requêtes MySQL complexes](../../tests-requetes-mysql-complexes/README.md) — stratégies complémentaires pour les requêtes complexes
- [Refactorisation code legacy](../../refactorisation-code-legacy/README.md) — stratégies de refactoring progressif

---

**Rappel** : L'approche actuelle (PHPUnit + données en base) n'est pas « fausse » — elle est incomplète. En ajoutant des tests unitaires sur la logique extraite et en stabilisant les tests d'intégration, on passe d'un filet de sécurité fragile à une suite de tests structurée, rapide et maintenable. Le tout sans refonte du module.
