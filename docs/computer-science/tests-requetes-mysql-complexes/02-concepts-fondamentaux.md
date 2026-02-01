# Concepts fondamentaux

Ce chapitre pose les bases : découpler l’exécution des requêtes du traitement des données, identifier les types de tests applicables, et garantir la reproductibilité des données.

## 1. Découplage requête / traitement

### Idée centrale

Une **requête complexe** produit un jeu de lignes (résultat SQL). Le **traitement** transforme ces lignes en objets métier, calcule des indicateurs, filtre ou agrège encore. En séparant clairement les deux responsabilités — une couche qui exécute la requête et retourne des lignes (ou DTOs bruts), une couche qui applique la logique métier sur ces données — on peut tester le traitement en unitaire en injectant un jeu de lignes fixe, sans base réelle.

En pratique : un **repository** (ou service d’accès données) exécute la requête et retourne une liste de lignes ou DTOs ; un **service métier** (ou mapper) prend cette liste en entrée et produit le résultat final. Les tests unitaires du service reçoivent alors des données en dur (ou mockées) ; les tests d’intégration vérifient que le repository + la requête produisent bien ces données pour un état de BDD donné.

### Limite

Certaines requêtes incluent déjà une logique métier forte (agrégations, expressions SQL). Dans ce cas, « tester la requête » et « tester le traitement » peuvent se confondre ; l’important est de savoir ce qu’on valide à chaque niveau (résultat SQL vs transformation applicative).

---

## 2. Types de tests applicables

### Test unitaire du traitement (données mockées)

- **Périmètre** : la logique qui transforme les lignes (mapping, calculs, règles). Pas de MySQL.
- **Données** : liste de lignes ou DTOs fournie en entrée (literal, factory, mock du repository).
- **Intérêt** : rapide, stable, ciblé. Couvre les régressions sur le traitement (nul, vide, cas limites).
- **Limite** : ne garantit pas que la requête produit effectivement ces données.

### Test d’intégration requête + BDD

- **Périmètre** : exécution réelle de la requête contre une base (locale, conteneur), puis éventuellement le traitement jusqu’au résultat final.
- **Données** : état de la base maîtrisé (fixtures, script d’initialisation, base dédiée ou isolée par test).
- **Intérêt** : valide la requête (JOINs, agrégations, ordre) et l’assemblage avec le driver et le schéma.
- **Limite** : plus lent, plus fragile si les données ne sont pas reproductibles.

### Test hybride (requête mockée, traitement réel)

- **Périmètre** : le service métier reçoit un résultat « comme si » venant du repository (mock qui retourne un jeu fixe) ; on teste uniquement le traitement.
- **Intérêt** : même idée que le unitaire du traitement, en passant par l’interface du repository ; utile si on ne veut pas exposer une fonction pure de transformation.

En résumé : **unitaire = traitement seul, avec données en entrée contrôlées ; intégration = requête + BDD (+ éventuellement traitement) avec données en base contrôlées.**

---

## 3. Reproductibilité des données

### Pourquoi c’est critique

Des tests d’intégration qui s’appuient sur des données partagées (autre environnement, autre développeur, données résiduelles) deviennent **non reproductibles** : un test passe ici et échoue ailleurs, ou inversement. Pour les requêtes complexes, un seul enregistrement en trop ou en moins peut changer le résultat ; l’ordre des tests peut aussi modifier l’état de la base.

### Principes

- **Isolation par test** : chaque test part d’un état connu (fixtures chargées au début du test ou de la classe) et, si besoin, nettoie ou recrée les données après exécution. Éviter les dépendances entre l’ordre des tests.
- **Fixtures dédiées** : jeux de données minimalement suffisants pour le scénario (ex. deux commandes, trois produits, un client). Éviter de reposer sur « ce qui existe déjà » en base.
- **Base dédiée ou conteneur** : en CI, utiliser une base créée pour les tests (Testcontainers MySQL, base de test isolée) plutôt qu’une base partagée avec d’autres usages.

### Compromis

- **Rollback de transaction** : exécuter chaque test dans une transaction qui est annulée à la fin ; la base revient à l’état initial. Simple mais attention aux autocommit et aux connexions.
- **Réinitialisation des fixtures** : avant chaque test (ou chaque classe), recréer les données (DELETE + INSERT ou script SQL). Plus coûteux, mais état garanti.
- **Base en mémoire (H2, SQLite mode MySQL)** : rapide, mais différences de dialecte possibles ; à réserver aux cas où la compatibilité MySQL stricte n’est pas critique, ou en complément des tests sur MySQL réelle.

---

## Synthèse rapide

| Niveau              | Périmètre                    | Données                    | Question typique                                      |
|---------------------|-----------------------------|----------------------------|--------------------------------------------------------|
| Unitaire traitement | Transformation des lignes   | Liste fixe / mock          | Ce traitement produit-il le bon résultat pour ces lignes ? |
| Intégration         | Requête + BDD (+ traitement)| Fixtures en base           | Cette requête + ce schéma produisent-ils le bon résultat ? |

---

## Prochaines étapes

- **[Stratégies et outils](./03-strategies-et-outils.md)** : Testcontainers, mocks du repository, assertions sur les jeux de données
- **[Exemples concrets](./04-exemples-concrets.md)** : rapports, agrégations, multi-tables, traitement après fetch
- **[Mise en pratique](./05-mise-en-pratique.md)** : quand quoi tester, pièges, CI

---

**Ressource** : [Martin Fowler — Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html) — même logique de niveaux appliquée à la couche données.
