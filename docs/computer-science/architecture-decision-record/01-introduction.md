# Introduction aux Architecture Decision Records

## Qu'est-ce qu'un Architecture Decision Record ?

Un **Architecture Decision Record** (ADR) est un document qui capture une décision architecturale importante, le contexte qui l'a motivée, et les conséquences de cette décision.

### Définition simple

Pensez à un ADR comme une **note de réunion** pour une décision importante, mais écrite de manière structurée et permanente. C'est la réponse à la question : "Pourquoi avons-nous fait ce choix technique ?"

### Exemple concret du quotidien

Imaginez que vous choisissez entre deux frameworks pour votre application web :

**Sans ADR** :
- 6 mois plus tard : "Pourquoi avons-nous choisi React au lieu de Vue ?"
- Réponse : "Je ne me souviens plus... Peut-être que quelqu'un l'aimait bien ?"

**Avec ADR** :
- 6 mois plus tard : Lecture de l'ADR #3
- Réponse claire : "Nous avons choisi React car notre équipe avait déjà de l'expérience, la communauté était plus grande pour notre cas d'usage, et nous avions besoin d'un écosystème mature pour les composants d'entreprise."

## Pourquoi les ADR sont-ils importants ?

### 1. La mémoire organisationnelle

**Problème** : Les décisions architecturales sont souvent prises lors de discussions informelles (Slack, réunions, conversations). Cette connaissance se perd avec le temps et le turnover.

**Solution** : Les ADR créent une mémoire permanente et accessible à tous.

**Exemple réel** :
```
Scénario : Une équipe choisit MongoDB pour stocker les logs.
3 ans plus tard : Un nouveau développeur demande "Pourquoi MongoDB ?"
Sans ADR : Personne ne se souvient, on assume que c'était un mauvais choix.
Avec ADR : On lit l'ADR et on comprend que MongoDB était choisi pour 
           sa scalabilité horizontale, nécessaire à l'époque.
```

### 2. Le contexte perdu

**Problème** : Les décisions ont du sens dans un contexte précis (contraintes budgétaires, compétences de l'équipe, délais). Ce contexte disparaît avec le temps.

**Solution** : Les ADR capturent le contexte au moment de la décision.

**Exemple concret** :
```
Décision : Utiliser une base de données SQL simple au lieu d'une solution NoSQL.
Contexte capturé dans l'ADR :
- Budget limité (pas de formation nécessaire)
- Équipe familière avec SQL
- Besoins de données relationnelles simples
- Délai serré (2 semaines)

Sans ce contexte, quelqu'un pourrait penser que c'était une erreur.
```

### 3. L'onboarding accéléré

**Problème** : Les nouveaux développeurs mettent des semaines à comprendre pourquoi l'architecture est structurée d'une certaine manière.

**Solution** : Les ADR servent de guide d'architecture pour les nouveaux arrivants.

**Exemple** :
```
Nouveau développeur : "Pourquoi avons-nous 3 bases de données différentes ?"
Réponse : "Lis les ADR #5, #12 et #18, ils expliquent chaque décision."
```

### 4. La révision des décisions

**Problème** : Les décisions prises il y a des années peuvent devenir obsolètes, mais personne ne sait si elles peuvent être révisées.

**Solution** : Les ADR permettent de réévaluer les décisions quand le contexte change.

**Exemple** :
```
ADR #7 (2019) : "Utiliser des serveurs physiques pour la base de données"
Contexte : Coûts cloud élevés à l'époque, besoin de contrôle total.

2024 : Les coûts cloud ont baissé, besoin de scalabilité automatique.
Action : Créer ADR #45 qui remplace ADR #7, migrer vers le cloud.
```

## Quand créer un ADR ?

### ✅ Créer un ADR pour...

#### Décisions technologiques majeures
- Choix d'un framework ou langage principal
- Choix d'une base de données
- Choix d'un système de messaging (RabbitMQ, Kafka, etc.)
- Choix d'une solution cloud (AWS, Azure, GCP)

**Exemple** :
```
Décision : Choisir PostgreSQL au lieu de MySQL
Impact : Toute l'équipe devra apprendre PostgreSQL
Durée : Cette décision affectera le projet pendant des années
→ ADR nécessaire
```

#### Décisions architecturales
- Architecture monolithique vs microservices
- Pattern de communication (REST, GraphQL, gRPC)
- Stratégie de déploiement (blue-green, canary)
- Pattern de données (CQRS, Event Sourcing)

**Exemple** :
```
Décision : Adopter une architecture microservices
Impact : Changement fondamental de l'organisation du code
Complexité : Augmentation significative de la complexité opérationnelle
→ ADR nécessaire
```

#### Décisions de sécurité
- Méthode d'authentification (OAuth, JWT, sessions)
- Stratégie de chiffrement
- Gestion des secrets
- Politique de sécurité des APIs

**Exemple** :
```
Décision : Utiliser OAuth 2.0 pour l'authentification
Impact : Tous les clients devront s'adapter
Sécurité : Impact direct sur la sécurité de l'application
→ ADR nécessaire
```

#### Décisions de performance
- Stratégie de cache (Redis, Memcached, cache applicatif)
- Optimisations majeures
- Choix de CDN
- Stratégie de mise à l'échelle

**Exemple** :
```
Décision : Implémenter Redis pour le cache distribué
Impact : Nouvelle dépendance infrastructure
Performance : Amélioration attendue de 50% sur les temps de réponse
→ ADR nécessaire
```

### ❌ Ne pas créer un ADR pour...

#### Décisions triviales
- Choix d'une bibliothèque utilitaire mineure
- Format de code (sauf si c'est une décision d'équipe majeure)
- Nommage de variables
- Décisions temporaires ou expérimentales

**Exemple** :
```
Décision : Utiliser lodash pour une fonction utilitaire
Impact : Aucun impact architectural
Durée : Peut être changé facilement
→ Pas besoin d'ADR
```

#### Décisions fonctionnelles
- Ajout d'une nouvelle feature
- Modification d'un workflow utilisateur
- Changements d'interface utilisateur

**Exemple** :
```
Décision : Ajouter un bouton "Exporter en PDF"
Impact : Fonctionnel uniquement, pas architectural
→ Pas besoin d'ADR (mais peut-être une user story)
```

## Les différents types d'ADR

### 1. ADR de décision (Decision ADR)
Le type le plus commun. Documente une décision qui a été prise.

**Exemple** : "ADR-001 : Utiliser React pour le frontend"

### 2. ADR de statut (Status ADR)
Documente le statut d'une décision (proposée, acceptée, rejetée, remplacée).

**Exemple** : "ADR-001 : Statut changé à 'Remplacé' par ADR-015"

### 3. ADR de remplacement (Replacement ADR)
Documente qu'une décision précédente est remplacée par une nouvelle.

**Exemple** : "ADR-015 : Remplacer React par Vue.js (remplace ADR-001)"

## Avantages et inconvénients

### ✅ Avantages

1. **Traçabilité** : Comprendre l'historique des décisions
2. **Communication** : Partager le contexte avec toute l'équipe
3. **Apprentissage** : Documenter les erreurs et les succès
4. **Onboarding** : Accélérer l'intégration des nouveaux développeurs
5. **Révision** : Faciliter la révision des décisions obsolètes

### ⚠️ Inconvénients à éviter

1. **Surcharge** : Créer trop d'ADR pour des décisions mineures
2. **Maintenance** : Les ADR doivent être maintenus à jour
3. **Temps** : Prend du temps à rédiger (mais fait gagner du temps plus tard)
4. **Résistance** : Certains développeurs peuvent voir ça comme de la bureaucratie

### Comment éviter les inconvénients ?

- **Se concentrer sur les décisions importantes** : Pas d'ADR pour tout
- **Format simple** : Utiliser un template pour aller vite
- **Intégrer dans le workflow** : Créer l'ADR pendant la décision, pas après
- **Faire court** : Un ADR doit être lisible en 5 minutes

## Comparaison avec d'autres approches

### ADR vs Documentation technique

| Aspect | Documentation technique | ADR |
|--------|------------------------|-----|
| **Focus** | Comment ça fonctionne | Pourquoi cette décision |
| **Détails** | Très détaillé | Concis et ciblé |
| **Mise à jour** | Doit être maintenue | Relativement stable |
| **Public** | Développeurs | Toute l'équipe |

### ADR vs Commentaires dans le code

| Aspect | Commentaires code | ADR |
|--------|-------------------|-----|
| **Visibilité** | Dans le code source | Accessible à tous |
| **Contexte** | Limité | Complet avec alternatives |
| **Historique** | Peut être modifié | Versionné dans Git |
| **Format** | Informel | Structuré |

## Conclusion

Les Architecture Decision Records sont un outil simple mais puissant pour :
- **Capturer** les décisions importantes
- **Communiquer** le contexte à toute l'équipe
- **Apprendre** des décisions passées
- **Réviser** les décisions quand le contexte change

Dans le prochain chapitre, nous verrons comment structurer et formater un ADR efficace.
