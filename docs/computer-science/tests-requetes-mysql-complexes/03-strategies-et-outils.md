# Stratégies et outils

Ce chapitre présente les **méthodes courantes et recommandées** pour tester les requêtes MySQL complexes et le traitement des données : BDD de test (conteneur, fixtures), mocks des résultats, et assertions sur les jeux de données.

## 1. Base de test : Testcontainers et MySQL

### Pourquoi une vraie MySQL en test

Les requêtes complexes (JOINs, agrégations, fonctions MySQL, types) peuvent se comporter différemment d’une base en mémoire (H2, SQLite). Pour éviter les faux positifs (« ça passe en test » mais échoue en prod à cause du dialecte), l’usage d’une **MySQL réelle** en test est souvent recommandé. Testcontainers lance un conteneur MySQL éphémère ; chaque run (ou chaque classe de tests) dispose d’une base propre.

### Utilisation type

- Démarrer le conteneur MySQL une fois par suite de tests (ou une fois par test si l’isolation l’exige).
- Appliquer le schéma (CREATE TABLE, etc.) et les fixtures (INSERTs ou scripts).
- Exécuter les tests d’intégration qui lancent les requêtes contre cette instance.
- Arrêter le conteneur à la fin.

Avantage : **reproductibilité** et **compatibilité** avec la MySQL de prod. Coût : temps de démarrage du conteneur, ressources ; en général on regroupe les tests d’intégration BDD pour ne pas multiplier les démarrages.

### Alternative : MySQL locale ou partagée

Certaines équipes utilisent une instance MySQL dédiée (locale ou sur un serveur de CI) avec un schéma et des fixtures réinitialisés avant les tests. Moins isolé qu’un conteneur par run, mais plus simple à mettre en place ; à réserver à des environnements maîtrisés pour éviter les interférences.

---

## 2. Fixtures et scripts d’initialisation

### Rôle des fixtures

Les **fixtures** définissent l’état de la base au début du test (ou de la suite). Pour une requête complexe, on insère le minimum nécessaire : par exemple deux clients, trois commandes, cinq lignes de détail, pour valider un rapport avec JOINs et agrégations. Sans fixtures explicites, les tests dépendent de données existantes ou de l’ordre d’exécution — source de flakiness.

### Formes courantes

- **Scripts SQL** : fichier `fixtures/orders_report.sql` avec INSERTs. Exécuté avant les tests (une fois par classe ou par test selon la stratégie).
- **Code (builders, factories)** : dans le langage applicatif, des helpers qui insèrent les données via le repository ou du SQL brut. Utile quand les données dépendent de règles métier ou d’IDs générés.
- **Outils de seed** : selon la stack (PHP: DbUnit, Laravel Seeders ; Java: Flyway/Liquibase + données de test ; etc.), utilisation des mêmes mécanismes que pour les environnements de dev, en version dédiée test.

Recommandation : **fixtures minimales et lisibles** — assez pour couvrir les cas (vide, un enregistrement, plusieurs avec relations), pas un copier-coller de la prod.

---

## 3. Mocker le résultat du repository (tests unitaires du traitement)

### Principe

Le **repository** (ou couche d’accès) est une dépendance du service qui fait le traitement. En test unitaire, on n’exécute pas la requête : on injecte un **mock** (ou stub) du repository qui retourne un jeu de lignes ou DTOs fixe. Le test vérifie que le service transforme correctement ces données (mapping, calculs, gestion du vide).

### Intérêt

- **Rapide** : pas de BDD, pas de réseau.
- **Ciblé** : un échec indique un problème dans la logique de traitement, pas dans la requête.
- **Cas limites** : on peut facilement fournir une liste vide, un nul, des doublons, pour vérifier le comportement.

### Mise en œuvre

- Interface ou contrat du repository : par ex. `findOrdersForReport(Filter f): List<OrderRow>`.
- En test : `when(repository.findOrdersForReport(any())).thenReturn(List.of(row1, row2));` puis appel du service et assertions sur le résultat.
- Les données `row1`, `row2` sont construites en dur ou via des builders ; elles représentent « ce que la requête serait censée retourner » pour le scénario.

Attention : **ne pas sur-mocker**. Si le traitement est trivial (simple mapping sans règle), un test unitaire peut apporter peu ; en revanche, dès qu’il y a calculs, filtrage ou règles métier sur les lignes, le unitaire avec données mockées est pertinent.

---

## 4. Assertions sur les jeux de données

### Comparer le résultat à une référence

Pour un test d’intégration, le résultat de la requête (ou du flux requête + traitement) est une structure de données : liste d’objets, tableau, JSON. Deux approches courantes :

- **Assertions ciblées** : vérifier les champs importants (nombre de lignes, valeur d’une colonne pour la première ligne, somme d’un agrégat). Moins fragile qu’un snapshot global ; en revanche, il faut choisir ce qui « compte » pour le scénario.
- **Snapshot** : comparer le résultat entier (sérialisé en JSON ou texte) à une version enregistrée. Détecte tout changement ; les évolutions légitimes obligent à mettre à jour le snapshot. À utiliser avec parcimonie (ex. rapport critique dont la structure ne change pas souvent).

### Ordre et stabilité

Les requêtes sans ORDER BY peuvent retourner les lignes dans un ordre non déterministe. Pour des assertions stables : soit **trier le résultat** avant comparaison (en test), soit **ne pas se fier à l’ordre** et vérifier des propriétés d’ensemble (taille, présence d’éléments, agrégats). En production, ajouter un ORDER BY explicite si l’ordre métier compte.

---

## 5. Résumé des stratégies recommandées

| Objectif                         | Stratégie recommandée                                      |
|----------------------------------|-------------------------------------------------------------|
| Tester la logique de traitement | Unitaire avec mock du repository (jeu de lignes fixe)     |
| Tester la requête SQL           | Intégration avec MySQL (Testcontainers ou base dédiée) + fixtures |
| Tester l’assemblage complet     | Intégration : requête + traitement contre BDD de test       |
| Reproductibilité                 | Fixtures minimales, base isolée (conteneur ou dédiée), pas de dépendance à l’ordre des tests |
| Stabilité des assertions        | Assertions ciblées sur champs clés ; si snapshot, le réserver à des cas précis |

---

## Prochaines étapes

- **[Exemples concrets](./04-exemples-concrets.md)** : rapport avec agrégation, requête multi-tables, traitement après fetch
- **[Mise en pratique](./05-mise-en-pratique.md)** : quand quoi tester, pièges, intégration dans le workflow et la CI

---

**Ressources** :

* [Testcontainers — MySQL](https://testcontainers.com/modules/databases/mysql/)
* [Martin Fowler — Unit Testing](https://martinfowler.com/testing/)
