# Mise en pratique

Ce guide aide à appliquer une stratégie de tests cohérente autour des requêtes MySQL complexes et du traitement des données : quand privilégier quelle approche, pièges courants et intégration dans le workflow et la CI.

## Quand privilégier quelle approche

### Privilégier les tests unitaires du traitement quand

- La **logique métier** sur les lignes (calculs, filtrage, agrégation applicative) est non triviale : fournir un jeu de lignes fixe (mock du repository) et vérifier le résultat.
- Vous voulez un **feedback rapide** et des cas limites (liste vide, nul, doublons) sans dépendre de la BDD.
- La requête est déjà couverte par un (ou quelques) tests d’intégration ; vous complétez par des unitaires sur le traitement.

### Privilégier les tests d’intégration (requête + BDD) quand

- La **requête** est complexe (JOINs multiples, agrégations, sous-requêtes) et vous devez valider qu’elle retourne le bon résultat pour un état de base donné.
- Vous modifiez le **schéma** ou la requête et voulez éviter les régressions silencieuses.
- Vous introduisez ou modifiez une **couche de persistance** (repository, mapping) et voulez vérifier l’assemblage avec une MySQL réelle.

En garder un nombre maîtrisé : plus lents et plus coûteux en ressources ; utiliser des fixtures minimales et une base isolée (Testcontainers ou dédiée).

### Réserver les tests « bout en bout » (requête + traitement + API) pour

- Quelques **scénarios critiques** (ex. endpoint de rapport utilisé par le front) pour valider le flux complet.
- La **confiance avant release** : pas besoin de couvrir chaque requête en E2E ; les tests d’intégration BDD + unitaires du traitement y contribuent déjà.

Trop de tests E2E sur des rapports alourdit le pipeline et multiplie la maintenance (données, timing). Mieux vaut peu de tests E2E ciblés que beaucoup de tests flaky.

---

## Pièges à éviter

### Données partagées ou dépendance à l’ordre

Des tests qui s’appuient sur des données existantes (autre environnement, résidus d’un test précédent) ou sur l’ordre d’exécution deviennent **non reproductibles**. Chaque test d’intégration BDD doit partir d’un état connu (fixtures chargées au début du test ou de la classe) et, si besoin, nettoyer ou recréer les données. Éviter les dépendances entre tests (test A crée des données, test B les utilise sans les recréer).

### Ne pas isoler les données par test

Lancer plusieurs tests d’intégration sur la même base sans réinitialiser les fixtures peut faire passer ou échouer un test à cause des données laissées par un autre. Stratégies courantes : transaction rollback par test, ou réinitialisation des fixtures avant chaque test (ou chaque classe). Choisir une stratégie et s’y tenir.

### Sur-mocker en unitaire

Tout mocker (y compris des objets de données simples) rend les tests verbeux et fragiles. Mocker le **repository** (ou la couche qui exécute la requête) et fournir un **jeu de lignes réaliste** suffit pour tester le traitement. Garder les données en entrée lisibles (builders, constantes) pour que le scénario reste clair.

### Assertions dépendantes de l’ordre ou du détail superflu

Les requêtes sans ORDER BY peuvent retourner les lignes dans un ordre non déterministe. Éviter des assertions du type « la première ligne est X » sans trier le résultat en test ou sans vérifier des propriétés d’ensemble (taille, présence d’éléments, agrégats). De même, éviter de comparer des champs qui ne sont pas pertinents pour le scénario (IDs techniques, timestamps non maîtrisés) si cela rend le test fragile.

### Négliger la compatibilité MySQL

Tester uniquement contre une base en mémoire (H2, SQLite) peut masquer des différences de dialecte (fonctions, types, comportement des JOINs). Pour les requêtes complexes, une **MySQL réelle** en test (Testcontainers ou instance dédiée) réduit les surprises en production. Certaines équipes gardent H2 pour des tests rapides et une suite plus petite contre MySQL en CI.

---

## Intégration dans le workflow

### Local

- **Unitaire (traitement)** : à chaque modification, idéalement en watch ou à chaque sauvegarde.
- **Intégration (requête + BDD)** : avant commit ou push, ou sur demande (script dédié). Si Testcontainers, le premier run peut être plus long (téléchargement de l’image).

### CI (pipeline)

- **Unitaire** : toujours, sur chaque commit ou PR ; échec = blocant.
- **Intégration BDD** : selon la durée et les ressources — souvent sur chaque PR ou sur la branche principale. Utiliser Testcontainers (ou une MySQL dédiée) pour reproductibilité ; s’assurer que les fixtures sont appliquées de façon déterministe.
- **E2E (rapports / flux complets)** : souvent sur la branche principale ou avant release ; parfois en parallèle pour ne pas bloquer le feedback rapide.

Adapter les seuils (nombre de tests d’intégration, durée maximale) au contexte projet : une équipe petite peut lancer toute la suite à chaque push ; une grosse codebase peut réserver les tests d’intégration BDD à des branches ciblées ou à la nuit.

---

## Questions à se poser

- **Qu’est-ce que ce test valide ?** (la requête, le traitement, les deux.) Cela guide le niveau (unitaire vs intégration).
- **Les données sont-elles reproductibles ?** (fixtures explicites, base isolée, pas de dépendance à l’ordre des tests.)
- **Le coût de maintenance est-il acceptable ?** (moins de tests d’intégration lourds, mocks ciblés pour le traitement.)
- **La CI reste-t-elle utilisable ?** (temps total, stabilité des tests, ressources pour Testcontainers.)

Dans votre contexte, le bon dosage dépend du nombre de requêtes complexes, de la criticité des rapports et de la tolérance au risque. Ajuster au fil du temps plutôt que d’appliquer une règle rigide.

---

## Ressources

- [Martin Fowler — Practical Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html)
- [Testcontainers — MySQL](https://testcontainers.com/modules/databases/mysql/)
- [Types de tests](../types-de-tests/README.md) — rappel des niveaux (unitaire, intégration, E2E)

---

**Rappel** : Tester les requêtes MySQL complexes et le traitement des données sert la confiance dans les changements et la maintenabilité. Découpler requête et traitement, maîtriser les données de test et choisir le bon niveau (unitaire vs intégration) permettent de garder des tests rapides, stables et ciblés.
