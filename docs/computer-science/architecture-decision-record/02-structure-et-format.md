# Structure et Format d'un ADR

## Format standard d'un ADR

Un ADR suit généralement une structure simple et cohérente. Il existe plusieurs formats, mais le plus courant est celui proposé par Michael Nygard, avec quelques variantes modernes.

## Structure de base (format Nygard)

### Format minimal

```markdown
# ADR-001 : [Titre de la décision]

## Statut
[Proposé | Accepté | Rejeté | Remplacé]

## Contexte
[Pourquoi cette décision est nécessaire]

## Décision
[Quelle décision a été prise]

## Conséquences
[Avantages et inconvénients]
```

### Format complet recommandé

```markdown
# ADR-001 : [Titre de la décision]

## Statut
[Proposé | Accepté | Rejeté | Remplacé | Dépublié]

## Contexte
[Description détaillée du problème ou de la situation]

## Décision
[La décision qui a été prise]

## Alternatives considérées
[Les autres options qui ont été évaluées]

## Conséquences
### Positives
- [Avantage 1]
- [Avantage 2]

### Négatives
- [Inconvénient 1]
- [Inconvénient 2]

### Neutres / À surveiller
- [Point d'attention]
```

## Détail de chaque section

### 1. Titre et numéro

**Format** : `ADR-XXX : [Titre descriptif et concis]`

**Règles** :
- Numéro séquentiel (001, 002, 003...)
- Titre court mais descriptif
- Utiliser des verbes d'action quand possible

**Exemples** :
- Bon : `ADR-001 : Utiliser React pour le frontend`
- Bon : `ADR-002 : Adopter PostgreSQL comme base de données principale`
- À éviter : `ADR-001 : Décision sur le framework` (trop vague)
- À éviter : `ADR-001 : React` (pas assez descriptif)

### 2. Statut

**Valeurs possibles** :
- **Proposé** : La décision est en cours de discussion
- **Accepté** : La décision a été approuvée et implémentée
- **Rejeté** : La décision a été rejetée (documenter pourquoi)
- **Remplacé** : La décision a été remplacée par un autre ADR
- **Dépublié** : La décision n'est plus pertinente (rare)

**Exemple** :
```markdown
## Statut
Accepté

## Note
Cet ADR remplace ADR-005 qui proposait Vue.js.
```

### 3. Contexte

**Objectif** : Expliquer **pourquoi** cette décision est nécessaire.

**Doit contenir** :
- Le problème à résoudre
- Les contraintes (techniques, budgétaires, temporelles)
- Les exigences (performance, sécurité, scalabilité)
- L'état actuel du système

**Exemple** :
```markdown
## Contexte

Notre application web actuelle utilise jQuery et du code JavaScript vanilla,
ce qui pose plusieurs problèmes :

1. **Maintenabilité** : Le code devient difficile à maintenir avec la croissance
2. **Performance** : Pas de système de composants réutilisables
3. **Recrutement** : Les développeurs modernes préfèrent les frameworks modernes
4. **Écosystème** : Manque d'outils et de bibliothèques modernes

Nous devons choisir un framework moderne pour le frontend qui :
- Soit facile à apprendre pour l'équipe actuelle
- Ait une grande communauté et un écosystème mature
- Soit performant et maintenable à long terme
- Supporte le TypeScript (requis par l'équipe)
```

### 4. Décision

**Objectif** : Énoncer clairement **quelle** décision a été prise.

**Doit être** :
- Claire et sans ambiguïté
- Spécifique (pas de "peut-être" ou "probablement")
- Actionnable

**Exemple** :
```markdown
## Décision

Nous utiliserons **React 18** comme framework frontend pour toutes les nouvelles
fonctionnalités et migrerons progressivement le code existant.

Nous utiliserons également :
- TypeScript pour le typage statique
- React Router pour la navigation
- Zustand pour la gestion d'état (léger, pas besoin de Redux)
```

### 5. Alternatives considérées

**Objectif** : Montrer que la décision a été réfléchie et documenter pourquoi les alternatives ont été rejetées.

**Format recommandé** : Liste avec pour chaque alternative :
- Description
- Avantages
- Inconvénients
- Pourquoi elle a été rejetée

**Exemple** :
```markdown
## Alternatives considérées

### 1. Vue.js 3
**Avantages** :
- Syntaxe simple et intuitive
- Excellente documentation
- Performance très bonne

**Inconvénients** :
- Écosystème moins large que React
- Moins de développeurs disponibles sur le marché

**Pourquoi rejeté** : L'équipe a déjà de l'expérience avec React, et nous
avons besoin d'un écosystème très large pour les composants d'entreprise.

### 2. Angular
**Avantages** :
- Framework complet (routing, state management inclus)
- TypeScript natif
- Bon pour les grandes applications

**Inconvénients** :
- Courbe d'apprentissage plus raide
- Plus verbeux que React
- Moins flexible

**Pourquoi rejeté** : Trop lourd pour nos besoins actuels, et l'équipe
préfère la flexibilité de React.

### 3. Svelte
**Avantages** :
- Performance exceptionnelle
- Syntaxe simple
- Bundle size très petit

**Inconvénients** :
- Écosystème encore jeune
- Moins de développeurs disponibles
- Moins de composants tiers

**Pourquoi rejeté** : Écosystème trop jeune pour nos besoins d'entreprise.
```

### 6. Conséquences

**Objectif** : Documenter les impacts de la décision.

**Doit couvrir** :
- Impacts positifs
- Impacts négatifs
- Points d'attention à surveiller
- Impacts sur d'autres parties du système

**Exemple** :
```markdown
## Conséquences

### Positives
- **Productivité** : Développement plus rapide grâce aux composants réutilisables
- **Maintenabilité** : Code plus structuré et plus facile à maintenir
- **Recrutement** : Plus facile d'attirer des développeurs React
- **Écosystème** : Accès à une vaste bibliothèque de composants (Material-UI, Ant Design)
- **Performance** : React optimise automatiquement le rendu avec le Virtual DOM

### Négatives
- **Courbe d'apprentissage** : L'équipe devra apprendre React (2-3 semaines)
- **Taille du bundle** : React ajoute ~40KB au bundle JavaScript
- **Migration** : Migration progressive du code existant nécessaire (6 mois estimés)
- **Complexité** : Introduction de nouveaux concepts (hooks, JSX, etc.)

### À surveiller
- **Performance** : Surveiller la taille du bundle et les temps de chargement
- **Formation** : S'assurer que toute l'équipe est formée correctement
- **Migration** : Planifier la migration du code legacy sans bloquer les nouvelles features
- **Compatibilité** : Vérifier la compatibilité avec nos outils existants (CI/CD, tests)
```

## Format avancé (avec métadonnées)

Pour des projets plus complexes, vous pouvez ajouter des métadonnées :

```markdown
# ADR-001 : Utiliser React pour le frontend

**Date** : 2024-01-15  
**Auteur(s)** : Alice Martin, Bob Dupont  
**Décideur(s)** : Équipe technique  
**Consulté(s)** : CTO, Lead Frontend  
**Tags** : frontend, framework, react, typescript

## Statut
Accepté

[... reste du contenu ...]
```

## Exemple complet d'ADR

Voici un exemple complet et réaliste :

```markdown
# ADR-003 : Utiliser PostgreSQL au lieu de MySQL

**Date** : 2024-01-20  
**Auteur** : Équipe Backend  
**Décideur** : Tech Lead  
**Tags** : database, postgresql, infrastructure

## Statut
Accepté

## Contexte

Notre application utilise actuellement MySQL 5.7 comme base de données principale.
Nous devons migrer vers une nouvelle version car MySQL 5.7 atteint sa fin de vie.

Nous avons besoin d'une base de données qui :
- Supporte les transactions ACID
- Gère bien les requêtes complexes avec JOINs
- Offre de bonnes performances pour les lectures et écritures
- Supporte le JSON natif (pour certains cas d'usage)
- A une bonne communauté et documentation
- Est compatible avec notre infrastructure cloud actuelle (AWS RDS)

## Décision

Nous migrerons vers **PostgreSQL 15** comme base de données principale.

La migration se fera en plusieurs étapes :
1. Mise en place d'une instance PostgreSQL en parallèle
2. Migration des données avec pgloader
3. Tests de régression complets
4. Bascule progressive (blue-green deployment)

## Alternatives considérées

### 1. MySQL 8.0
**Avantages** :
- Migration plus simple (même famille)
- L'équipe connaît déjà MySQL
- Compatible avec notre code existant

**Inconvénients** :
- Moins de fonctionnalités avancées (JSON, arrays)
- Performance inférieure pour les requêtes complexes
- Licence Oracle (préoccupations à long terme)

**Pourquoi rejeté** : Nous avons besoin de fonctionnalités avancées (JSON natif,
full-text search amélioré) et PostgreSQL offre de meilleures performances
pour nos cas d'usage.

### 2. MongoDB
**Avantages** :
- Très flexible (schema-less)
- Excellente scalabilité horizontale
- Bon pour les données non-structurées

**Inconvénients** :
- Pas de transactions multi-documents fiables (à l'époque)
- Requiert un changement complet de modèle de données
- L'équipe n'a pas d'expérience avec NoSQL

**Pourquoi rejeté** : Nos données sont fortement relationnelles et nous avons
besoin de transactions ACID. Le changement serait trop risqué et coûteux.

### 3. Amazon Aurora PostgreSQL
**Avantages** :
- Gestion complète par AWS
- Scalabilité automatique
- Backups automatiques

**Inconvénients** :
- Coût plus élevé que RDS standard
- Vendor lock-in avec AWS
- Pas nécessaire pour notre taille actuelle

**Pourquoi rejeté** : Nous commençons avec RDS PostgreSQL standard. Nous
réévaluerons Aurora quand nous aurons besoin de plus de scalabilité.

## Conséquences

### Positives
- **Fonctionnalités** : Accès à des fonctionnalités avancées (JSON natif, arrays, full-text search)
- **Performance** : Meilleures performances pour les requêtes complexes
- **Standard SQL** : Meilleure conformité aux standards SQL
- **Écosystème** : Large écosystème d'extensions (PostGIS, pg_trgm, etc.)
- **Open source** : Licence PostgreSQL (plus permissive que MySQL)

### Négatives
- **Migration** : Migration complexe nécessitant des tests approfondis (2 mois estimés)
- **Formation** : L'équipe devra apprendre les spécificités de PostgreSQL
- **Coûts** : Coûts de migration et de formation
- **Risque** : Risque de bugs pendant la migration

### À surveiller
- **Performance** : Surveiller les performances après migration
- **Compatibilité** : Vérifier la compatibilité de toutes les requêtes SQL
- **Formation** : S'assurer que l'équipe est formée aux différences MySQL/PostgreSQL
- **Backup** : Mettre en place une stratégie de backup robuste
- **Monitoring** : Configurer le monitoring spécifique à PostgreSQL

## Notes de migration

- Utiliser `pgloader` pour la migration initiale des données
- Créer un script de validation pour comparer les données MySQL et PostgreSQL
- Planifier une fenêtre de maintenance de 4 heures pour la bascule
- Avoir un plan de rollback prêt (garder MySQL en parallèle pendant 1 mois)
```

## Bonnes pratiques de rédaction

### 1. Être concis mais complet

**À éviter : Trop court** :
```markdown
## Contexte
On a besoin d'une nouvelle DB.

## Décision
PostgreSQL.
```

**Bon** :
```markdown
## Contexte
MySQL 5.7 atteint sa fin de vie. Nous devons migrer vers une nouvelle version.
Nos besoins : transactions ACID, requêtes complexes, JSON natif.

## Décision
PostgreSQL 15 pour ses fonctionnalités avancées et meilleures performances.
```

### 2. Utiliser un langage clair

- Éviter le jargon technique excessif
- Expliquer les acronymes la première fois
- Utiliser des exemples concrets

### 3. Être honnête sur les compromis

- Documenter les inconvénients, pas seulement les avantages
- Reconnaître les risques et incertitudes
- Être transparent sur les compromis

### 4. Maintenir la cohérence

- Utiliser le même format pour tous les ADR
- Suivre la numérotation séquentielle
- Utiliser des tags cohérents

## Organisation des fichiers

### Structure de dossiers recommandée

```
docs/
  adr/
    0001-utiliser-react-frontend.md
    0002-adopter-postgresql.md
    0003-implementer-authentification-oauth.md
    README.md (index de tous les ADR)
```

### Nommage des fichiers

**Format recommandé** : `NNNN-titre-en-minuscules.md`

**Exemples** :
- `0001-utiliser-react-frontend.md`
- `0002-adopter-postgresql.md`
- `0015-remplacer-react-par-vue.md`

**Avantages** :
- Tri naturel dans l'explorateur de fichiers
- Facile à référencer
- Compatible avec Git

### Fichier README.md (index)

Créer un fichier `README.md` qui liste tous les ADR :

```markdown
# Architecture Decision Records

## Liste des ADR

- [ADR-001 : Utiliser React pour le frontend](./0001-utiliser-react-frontend.md) - Accepté
- [ADR-002 : Adopter PostgreSQL](./0002-adopter-postgresql.md) - Accepté
- [ADR-003 : Implémenter l'authentification OAuth](./0003-implementer-authentification-oauth.md) - Accepté
- [ADR-004 : Utiliser Docker pour le déploiement](./0004-utiliser-docker-deploiement.md) - Proposé
```

## Outils et templates

### Template Markdown réutilisable

Créer un fichier `template.md` :

```markdown
# ADR-XXX : [Titre]

**Date** : YYYY-MM-DD  
**Auteur(s)** :  
**Décideur(s)** :  
**Tags** :  

## Statut
[Proposé | Accepté | Rejeté | Remplacé]

## Contexte


## Décision


## Alternatives considérées

### 1. [Alternative 1]
**Avantages** :

**Inconvénients** :

**Pourquoi rejeté** :

## Conséquences

### Positives


### Négatives


### À surveiller


## Notes
```

### Outils utiles

- **adr-tools** : Outil en ligne de commande pour gérer les ADR
- **ADR Viewer** : Visualiseur d'ADR pour GitHub
- **Templates** : Créer des templates dans votre éditeur favori

## Conclusion

Un ADR bien structuré doit être :
- **Clair** : Facile à comprendre
- **Complet** : Contient toutes les informations nécessaires
- **Concis** : Pas trop long (idéalement 1-2 pages)
- **Actionnable** : La décision est claire
- **Traçable** : On peut comprendre le contexte et les alternatives

Dans le prochain chapitre, nous verrons des exemples concrets d'ADR dans différents contextes.
