# Mise en pratique des types de tests

Ce guide aide à appliquer une stratégie de tests cohérente : quand privilégier quel type, pièges courants et intégration dans le workflow.

## Quand écrire quel type de test

### Privilégier les tests unitaires quand

- La **logique métier** est pure (calculs, règles, transformations) : pas de BDD ni de réseau.
- Vous voulez un **feedback rapide** à chaque changement : des centaines de tests en quelques secondes.
- Vous avez des **cas limites** à couvrir (nul, vide, erreurs) sans dépendre de l’infra.

### Privilégier les tests d’intégration quand

- Vous devez vérifier que **plusieurs composants fonctionnent ensemble** : service + repository + BDD, ou contrôleur + service + BDD.
- Les **contrats** (SQL, mapping, API interne) sont critiques et peuvent casser sans que les unitaires ne le voient.
- Vous introduisez ou modifiez une **couche de persistance** ou une API interne.

En garder un nombre maîtrisé : ils sont plus lents et plus fragiles. Certaines équipes les lancent uniquement en CI ou sur une branche dédiée.

### Réserver les tests E2E pour

- **Quelques scénarios critiques** : connexion, commande, inscription, parcours métier clé.
- La **confiance avant release** : pas besoin de couvrir chaque cas limite en E2E ; les unitaires et l’intégration y contribuent déjà.

Trop d’E2E alourdit le pipeline et multiplie la maintenance (sélecteurs, timing, données). Mieux vaut peu d’E2E stables que beaucoup de tests flaky.

### Tests contractuels

- Utiles quand **plusieurs équipes ou services** évoluent indépendamment et partagent une API ou des messages.
- Permettent de détecter les **casses de contrat** sans déployer tout le système.

---

## Pièges à éviter

### Sur-mocker en unitaire

Tout mocker (y compris des objets de données ou des collaborateurs simples) rend les tests verbeux et fragiles : chaque refactoring casse des attentes sur les appels. Mocker les **dépendances externes** (repository, client HTTP, notification) suffit ; garder le reste réel quand c’est possible.

### Confondre unitaire et intégration

Un test qui lance une vraie BDD ou un vrai serveur HTTP n’est pas unitaire, même s’il est dans le même dossier. Clarifier le périmètre évite les surprises (lenteur, flakiness) et permet de choisir où l’exécuter (local vs CI).

### Négliger la stabilité des E2E

Tests E2E qui échouent aléatoirement (timing, ordre, données partagées) font perdre la confiance. Investir dans des **sélecteurs stables**, des **données de test dédiées** et des **attentes explicites** (attendre un élément visible plutôt qu’un délai fixe) réduit le flakiness.

### Viser la couverture pour elle-même

Une couverture à 100 % ne garantit pas que les cas à risque sont testés. Mieux vaut **cibler les chemins critiques et les régressions** que d’ajouter des tests pour faire monter un pourcentage.

### Tout tester en E2E

Répéter en E2E ce qui est déjà couvert en unitaire ou en intégration coûte cher en temps et en maintenance. Réserver l’E2E aux parcours utilisateur qui apportent une confiance que les autres niveaux ne donnent pas.

---

## Intégration dans le workflow

### Local

- **Unitaire** : à chaque modification, idéalement en watch ou à chaque sauvegarde.
- **Intégration** : avant commit ou push, ou sur demande (script dédié).
- **E2E** : sur demande ou avant merge, selon la durée et la stabilité.

### CI (pipeline)

- **Unitaire** : toujours, sur chaque commit ou PR ; échec = blocant.
- **Intégration** : selon la durée et les ressources (toujours, ou sur des branches ciblées). BDD en conteneur (Testcontainers, etc.) pour reproductibilité.
- **E2E** : souvent sur la branche principale ou avant release ; parfois en parallèle avec des tests plus rapides pour ne pas bloquer le feedback.

Adapter les seuils (nombre, durée, criticité) au contexte projet : une équipe petite avec peu de E2E peut les lancer à chaque push ; une grosse codebase peut les réserver à la nuit ou aux tags.

---

## Questions à se poser

- **Quel niveau répond à ma question ?** (logique seule → unitaire ; assemblage → intégration ; parcours utilisateur → E2E.)
- **Ce test va-t-il rester stable ?** (données dédiées, pas de dépendance à l’heure ou à l’ordre.)
- **Le coût de maintenance est-il acceptable ?** (moins de E2E complexes, mocks ciblés.)
- **Le pipeline reste-t-il utilisable ?** (temps total, échecs flaky.)

Dans votre contexte, le bon dosage dépend du domaine, de la taille de l’équipe et de la tolérance au risque. Ajuster au fil du temps plutôt que d’appliquer une règle rigide.

---

## Ressources

- [Martin Fowler — Practical Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html)
- [Google Testing Blog](https://testing.googleblog.com/) — bonnes pratiques et anti-patterns
- [Testcontainers](https://www.testcontainers.org/) — BDD et services en conteneurs pour les tests d’intégration

---

**Rappel** : Les types de tests servent la confiance et la maintenabilité à un coût maîtrisé. Choisir le bon niveau pour la bonne question et garder les tests stables et ciblés.
