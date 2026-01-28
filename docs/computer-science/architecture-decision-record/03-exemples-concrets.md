# Exemples Concrets d'ADR

Ce chapitre présente des exemples complets et réalistes d'Architecture Decision Records dans différents contextes. Chaque exemple illustre une situation réelle que vous pourriez rencontrer dans vos projets.

## Exemple 1 : Choix d'un Framework Frontend

### ADR-001 : Utiliser React pour le frontend

**Date** : 2024-01-15  
**Auteur** : Équipe Frontend  
**Décideur** : Tech Lead  
**Tags** : frontend, framework, react, typescript

#### Statut
Accepté

#### Contexte

Notre application web actuelle utilise jQuery et du code JavaScript vanilla écrit il y a 5 ans. Avec la croissance de l'application, nous rencontrons plusieurs problèmes :

1. **Maintenabilité** : Le code devient difficile à maintenir. Les modifications dans une partie du code cassent souvent d'autres parties.
2. **Performance** : Pas de système de composants réutilisables, ce qui entraîne de la duplication de code et des problèmes de performance.
3. **Recrutement** : Les développeurs modernes préfèrent travailler avec des frameworks modernes. Il devient difficile de recruter.
4. **Écosystème** : Manque d'accès aux outils et bibliothèques modernes (state management, routing, etc.).
5. **TypeScript** : L'équipe souhaite utiliser TypeScript pour améliorer la qualité du code.

Nous devons choisir un framework moderne pour le frontend qui :
- Soit facile à apprendre pour l'équipe actuelle (5 développeurs JavaScript)
- Ait une grande communauté et un écosystème mature
- Soit performant et maintenable à long terme
- Supporte le TypeScript nativement
- Permette une migration progressive (pas de réécriture complète d'un coup)

#### Décision

Nous utiliserons **React 18** comme framework frontend pour toutes les nouvelles fonctionnalités et migrerons progressivement le code existant.

Stack technique choisie :
- **React 18** : Framework UI
- **TypeScript** : Typage statique
- **React Router v6** : Navigation
- **Zustand** : Gestion d'état (léger, pas besoin de Redux pour notre cas)
- **Vite** : Build tool (plus rapide que Create React App)
- **Vitest** : Tests unitaires

Plan de migration :
1. Phase 1 (Mois 1-2) : Setup de la nouvelle stack, formation de l'équipe
2. Phase 2 (Mois 3-6) : Développement des nouvelles features en React
3. Phase 3 (Mois 7-12) : Migration progressive des pages existantes

#### Alternatives considérées

##### 1. Vue.js 3
**Avantages** :
- Syntaxe simple et intuitive (template-based)
- Excellente documentation officielle
- Performance très bonne (Composition API)
- Écosystème mature
- TypeScript bien supporté

**Inconvénients** :
- Écosystème moins large que React (moins de composants tiers)
- Moins de développeurs disponibles sur le marché français
- Moins utilisé dans les grandes entreprises (notre secteur)

**Pourquoi rejeté** : 
- L'équipe a déjà quelques développeurs avec de l'expérience React
- Nous avons besoin d'un écosystème très large pour les composants d'entreprise (tableaux de données, formulaires complexes)
- Le marché du recrutement favorise React dans notre région

##### 2. Angular
**Avantages** :
- Framework complet (routing, state management, forms inclus)
- TypeScript natif (pas de configuration)
- Bon pour les très grandes applications
- Structure très claire et opinionated
- Excellent pour les équipes nombreuses

**Inconvénients** :
- Courbe d'apprentissage plus raide
- Plus verbeux que React (plus de code boilerplate)
- Moins flexible (plus opinionated)
- Bundle size plus important
- Écosystème moins dynamique

**Pourquoi rejeté** : 
- Trop lourd pour nos besoins actuels (équipe de 5 personnes)
- L'équipe préfère la flexibilité de React
- Nous n'avons pas besoin de toutes les fonctionnalités incluses (overkill)

##### 3. Svelte / SvelteKit
**Avantages** :
- Performance exceptionnelle (compilation à la build)
- Syntaxe très simple et intuitive
- Bundle size très petit
- Pas de Virtual DOM (meilleures performances)

**Inconvénients** :
- Écosystème encore jeune comparé à React/Vue
- Moins de développeurs disponibles
- Moins de composants tiers disponibles
- TypeScript support moins mature

**Pourquoi rejeté** : 
- Écosystème trop jeune pour nos besoins d'entreprise
- Risque de manquer de ressources en cas de problème
- Moins de support communautaire pour les cas d'usage complexes

##### 4. Rester sur jQuery / Vanilla JS
**Avantages** :
- Pas de migration nécessaire
- L'équipe connaît déjà
- Pas de nouvelles dépendances

**Inconvénients** :
- Problèmes de maintenabilité continueront
- Difficultés de recrutement
- Pas d'accès aux outils modernes
- Performance sous-optimale

**Pourquoi rejeté** : 
- Les problèmes actuels ne seront pas résolus
- Risque de dette technique croissante
- Impact négatif sur le recrutement

#### Conséquences

##### Positives
- **Productivité** : Développement plus rapide grâce aux composants réutilisables et à l'écosystème
- **Maintenabilité** : Code plus structuré et plus facile à maintenir avec les composants React
- **Recrutement** : Plus facile d'attirer des développeurs React (marché plus large)
- **Écosystème** : Accès à une vaste bibliothèque de composants (Material-UI, Ant Design, React Table)
- **Performance** : React optimise automatiquement le rendu avec le Virtual DOM
- **TypeScript** : Meilleure qualité de code grâce au typage statique
- **Tests** : Meilleure testabilité avec les composants isolés

##### Négatives
- **Courbe d'apprentissage** : L'équipe devra apprendre React (2-3 semaines de formation)
- **Taille du bundle** : React ajoute ~40KB au bundle JavaScript (minifié + gzippé)
- **Migration** : Migration progressive du code existant nécessaire (6-12 mois estimés)
- **Complexité** : Introduction de nouveaux concepts (hooks, JSX, state management)
- **Coûts** : Formation de l'équipe et temps de migration
- **Risque** : Risque de bugs pendant la période de migration (deux systèmes en parallèle)

##### À surveiller
- **Performance** : Surveiller la taille du bundle et les temps de chargement (objectif : < 3s)
- **Formation** : S'assurer que toute l'équipe est formée correctement (budget formation alloué)
- **Migration** : Planifier la migration du code legacy sans bloquer les nouvelles features
- **Compatibilité** : Vérifier la compatibilité avec nos outils existants (CI/CD, tests E2E)
- **Bundle size** : Surveiller la croissance du bundle et utiliser le code splitting si nécessaire

#### Notes de mise en œuvre

- Créer un guide de style React pour l'équipe
- Mettre en place des code reviews pour garantir la qualité
- Organiser des sessions de pair programming pour la formation
- Créer une bibliothèque de composants partagés dès le début

---

## Exemple 2 : Architecture Microservices

### ADR-005 : Adopter une architecture microservices

**Date** : 2024-02-10  
**Auteur** : Équipe Architecture  
**Décideur** : CTO  
**Tags** : architecture, microservices, scalability, infrastructure

#### Statut
Accepté

#### Contexte

Notre application est actuellement un monolithe qui gère :
- Gestion des utilisateurs et authentification
- Catalogue de produits
- Commandes et paiements
- Système de notifications
- Reporting et analytics

Avec la croissance de l'entreprise, nous rencontrons plusieurs problèmes :

1. **Scalabilité** : Nous devons scaler toute l'application même si seule une partie a besoin de plus de ressources
2. **Déploiements** : Un changement dans une petite fonctionnalité nécessite de redéployer toute l'application
3. **Équipes** : Les équipes se marchent dessus car tout le monde travaille sur le même codebase
4. **Technologies** : Impossible d'utiliser différentes technologies pour différentes parties (ex: Python pour ML, Go pour performance)
5. **Fiabilité** : Un bug dans une partie peut faire planter toute l'application
6. **Performance** : Certaines parties sont lentes et impactent toute l'application

Nous avons besoin d'une architecture qui :
- Permette de scaler indépendamment chaque partie
- Permette des déploiements indépendants
- Permette à différentes équipes de travailler indépendamment
- Améliore la fiabilité (isolation des erreurs)
- Supporte différentes technologies si nécessaire

#### Décision

Nous adopterons une **architecture microservices** avec les principes suivants :

**Services identifiés** :
1. **User Service** : Gestion des utilisateurs et authentification
2. **Product Service** : Catalogue de produits
3. **Order Service** : Gestion des commandes
4. **Payment Service** : Traitement des paiements
5. **Notification Service** : Envoi de notifications (email, SMS, push)
6. **Analytics Service** : Reporting et analytics

**Stack technique** :
- **API Gateway** : Kong pour router les requêtes
- **Communication** : REST APIs pour la synchronisation, Message Queue (RabbitMQ) pour l'asynchrone
- **Base de données** : Chaque service a sa propre base de données (Database per Service pattern)
- **Service Discovery** : Consul pour la découverte de services
- **Monitoring** : Prometheus + Grafana
- **Logging** : ELK Stack (Elasticsearch, Logstash, Kibana)
- **Containerisation** : Docker + Kubernetes pour l'orchestration

**Plan de migration** :
1. Phase 1 (Mois 1-3) : Extraire le User Service (le plus indépendant)
2. Phase 2 (Mois 4-6) : Extraire le Product Service
3. Phase 3 (Mois 7-9) : Extraire Order et Payment Services
4. Phase 4 (Mois 10-12) : Extraire Notification et Analytics Services

#### Alternatives considérées

##### 1. Rester sur le monolithe
**Avantages** :
- Pas de migration nécessaire
- Plus simple à développer et débugger
- Pas besoin de gérer la complexité distribuée
- Transactions ACID faciles

**Inconvénients** :
- Scalabilité limitée (doit scaler tout)
- Déploiements risqués (tout ou rien)
- Équipes qui se bloquent mutuellement
- Impossible d'utiliser différentes technologies

**Pourquoi rejeté** : 
- Les problèmes actuels ne seront pas résolus
- Limite notre capacité de croissance
- Impact négatif sur la vélocité des équipes

##### 2. Architecture modulaire (Modular Monolith)
**Avantages** :
- Plus simple que les microservices
- Modules indépendants mais dans la même app
- Déploiements plus faciles que microservices
- Pas de complexité réseau distribuée

**Inconvénients** :
- Toujours un seul déploiement
- Scalabilité toujours limitée
- Partage de la même base de données (couplage)
- Même technologie pour tout

**Pourquoi rejeté** : 
- Ne résout pas complètement nos problèmes de scalabilité
- Ne permet pas l'indépendance complète des équipes
- Étape intermédiaire qui nécessitera une migration vers microservices plus tard

##### 3. Architecture Serverless (AWS Lambda, etc.)
**Avantages** :
- Scalabilité automatique
- Pas de gestion d'infrastructure
- Pay-per-use (coûts optimisés)
- Déploiements très simples

**Inconvénients** :
- Vendor lock-in (AWS, Azure, GCP)
- Cold starts (latence)
- Limitations de temps d'exécution
- Debugging plus complexe
- Coûts peuvent exploser à grande échelle

**Pourquoi rejeté** : 
- Vendor lock-in trop important pour notre stratégie
- Cold starts inacceptables pour certaines parties (paiements)
- Besoin de plus de contrôle sur l'infrastructure

#### Conséquences

##### Positives
- **Scalabilité indépendante** : Chaque service peut scaler selon ses besoins
- **Déploiements indépendants** : Déployer un service sans affecter les autres
- **Équipes autonomes** : Chaque équipe peut travailler sur son service indépendamment
- **Choix technologiques** : Possibilité d'utiliser différentes technologies (ex: Python pour ML, Go pour performance)
- **Fiabilité** : Isolation des erreurs (un bug dans un service n'affecte pas les autres)
- **Performance** : Optimisation indépendante de chaque service

##### Négatives
- **Complexité** : Architecture distribuée beaucoup plus complexe
- **Latence réseau** : Communication entre services ajoute de la latence
- **Gestion des données** : Pas de transactions distribuées ACID (doit utiliser Saga pattern)
- **Debugging** : Plus difficile de tracer les requêtes à travers plusieurs services
- **Coûts infrastructure** : Plus de ressources nécessaires (plus de serveurs, monitoring, etc.)
- **Opérations** : Besoin d'une équipe DevOps dédiée
- **Migration** : Migration complexe et risquée (12 mois estimés)

##### À surveiller
- **Latence** : Surveiller la latence entre services (objectif : < 100ms)
- **Disponibilité** : Chaque service doit avoir une disponibilité élevée (99.9%)
- **Monitoring** : Mettre en place un monitoring complet (traces distribuées avec Jaeger)
- **Gestion des erreurs** : Implémenter des patterns de résilience (circuit breaker, retry)
- **Sécurité** : Sécuriser la communication entre services (mTLS)
- **Coûts** : Surveiller les coûts d'infrastructure (peut être 2-3x plus cher)
- **Formation** : Former l'équipe aux patterns microservices (Saga, CQRS, etc.)

#### Notes de mise en œuvre

- Commencer petit : extraire un seul service d'abord pour apprendre
- Mettre en place l'infrastructure de base (monitoring, logging) avant la migration
- Créer des guides et standards pour les développeurs
- Organiser des sessions de formation sur les patterns microservices
- Mettre en place un service mesh (Istio) pour gérer la communication entre services

---

## Exemple 3 : Authentification et Autorisation

### ADR-008 : Implémenter l'authentification OAuth 2.0 avec JWT

**Date** : 2024-03-05  
**Auteur** : Équipe Sécurité  
**Décideur** : Security Lead  
**Tags** : security, authentication, oauth, jwt, api

#### Statut
Accepté

#### Contexte

Notre application actuelle utilise des sessions serveur (cookies) pour l'authentification. Avec l'évolution de notre architecture, nous rencontrons plusieurs problèmes :

1. **APIs multiples** : Nous avons maintenant plusieurs APIs (web, mobile, partenaires) qui doivent toutes authentifier les utilisateurs
2. **Scalabilité** : Les sessions serveur nécessitent un stockage partagé (Redis) et compliquent la scalabilité horizontale
3. **Mobile** : Les applications mobiles ont des difficultés avec les cookies
4. **Stateless** : Besoin d'une authentification stateless pour les microservices
5. **SSO** : Besoin de permettre l'authentification unique (SSO) avec des partenaires
6. **Sécurité** : Les sessions peuvent être vulnérables aux attaques CSRF

Nous avons besoin d'une solution d'authentification qui :
- Fonctionne pour web, mobile et APIs
- Soit stateless (pas de stockage serveur)
- Supporte le SSO avec des partenaires
- Soit sécurisée et conforme aux standards
- Permette la révocation de tokens
- Soit scalable

#### Décision

Nous implémenterons **OAuth 2.0 avec JWT (JSON Web Tokens)** comme mécanisme d'authentification.

**Architecture choisie** :
- **Authorization Server** : Service dédié qui émet les tokens (Keycloak ou Auth0)
- **JWT Tokens** : Access tokens (courte durée, 15 min) et Refresh tokens (longue durée, 7 jours)
- **Token Storage** : Access tokens dans le localStorage (web) ou Keychain (mobile), Refresh tokens en httpOnly cookies
- **API Gateway** : Validation des tokens JWT à l'entrée
- **Revocation** : Blacklist des tokens révoqués dans Redis (pour les access tokens) + invalidation des refresh tokens en base

**Flux d'authentification** :
1. Utilisateur se connecte → Authorization Server
2. Authorization Server valide les credentials → émet Access Token + Refresh Token
3. Client envoie Access Token dans le header `Authorization: Bearer <token>`
4. APIs valident le token JWT (signature + expiration)
5. Si Access Token expiré → utiliser Refresh Token pour en obtenir un nouveau

**Sécurité** :
- Tokens signés avec RS256 (asymétrique)
- HTTPS obligatoire
- Refresh tokens rotés à chaque utilisation
- Rate limiting sur les endpoints d'authentification
- Protection contre les attaques de rejeu (nonce dans les tokens)

#### Alternatives considérées

##### 1. Rester sur les sessions serveur
**Avantages** :
- Simple et bien compris par l'équipe
- Facile à révoquer (supprimer la session)
- Pas de problème de taille (les tokens JWT peuvent être gros)

**Inconvénients** :
- Nécessite un stockage partagé (Redis) pour la scalabilité
- Problèmes avec les apps mobiles (cookies)
- Pas adapté aux microservices (couplage)
- Vulnérable aux attaques CSRF

**Pourquoi rejeté** : 
- Ne répond pas à nos besoins (mobile, microservices, SSO)
- Complexité de gestion du stockage partagé

##### 2. API Keys simples
**Avantages** :
- Très simple à implémenter
- Pas de complexité OAuth
- Facile à révoquer

**Inconvénients** :
- Pas de standard
- Pas de gestion fine des permissions
- Pas de support SSO
- Sécurité limitée (si la clé est compromise)

**Pourquoi rejeté** : 
- Pas assez sécurisé pour notre cas d'usage
- Pas de support pour SSO avec partenaires
- Pas de standard industriel

##### 3. OAuth 2.0 avec tokens opaques
**Avantages** :
- Tokens plus petits (juste un ID)
- Facile à révoquer (supprimer de la base)
- Pas de problème de taille

**Inconvénients** :
- Nécessite une base de données pour valider chaque requête
- Latence ajoutée (requête DB à chaque validation)
- Pas stateless (couplage avec la DB)
- Scalabilité limitée

**Pourquoi rejeté** : 
- Impact sur les performances (requête DB à chaque requête API)
- Pas vraiment stateless
- Complexité opérationnelle (gestion de la DB de tokens)

##### 4. SAML 2.0
**Avantages** :
- Standard pour SSO entreprise
- Très sécurisé
- Bien supporté par les entreprises

**Inconvénients** :
- Complexe (XML, signatures, etc.)
- Principalement pour SSO entreprise, pas pour APIs
- Pas adapté aux applications mobiles
- Overkill pour notre cas

**Pourquoi rejeté** : 
- Trop complexe pour nos besoins
- Pas adapté aux APIs REST et mobile
- Principalement pour SSO entreprise (nous avons besoin de plus)

#### Conséquences

##### Positives
- **Stateless** : Pas besoin de stockage serveur pour les sessions
- **Scalabilité** : Facilement scalable (pas de stockage partagé)
- **Multi-plateforme** : Fonctionne pour web, mobile et APIs
- **SSO** : Support natif pour SSO avec partenaires (OAuth 2.0)
- **Standard** : OAuth 2.0 est un standard industriel
- **Flexibilité** : Tokens peuvent contenir des claims personnalisés (permissions, etc.)
- **Performance** : Validation locale des tokens (pas de requête DB)

##### Négatives
- **Complexité** : OAuth 2.0 est complexe à implémenter correctement
- **Taille des tokens** : Les JWT peuvent être gros (limite de taille des headers)
- **Révocation** : Plus difficile à révoquer qu'une session (nécessite une blacklist)
- **Sécurité** : Si un token est compromis, il est valide jusqu'à expiration
- **Courbe d'apprentissage** : L'équipe doit apprendre OAuth 2.0 et JWT
- **Coûts** : Si utilisation d'un service tiers (Auth0), coûts mensuels

##### À surveiller
- **Sécurité** : Surveiller les tentatives d'attaque (tokens volés, replay attacks)
- **Performance** : Surveiller le temps de validation des tokens
- **Taille des tokens** : Surveiller la taille des JWT (ne pas dépasser 8KB)
- **Expiration** : Taux d'utilisation des refresh tokens (si trop élevé, ajuster la durée)
- **Révocation** : Taille de la blacklist Redis (nettoyage automatique des tokens expirés)
- **Erreurs** : Taux d'erreurs d'authentification (tokens invalides, expirés)

#### Notes de mise en œuvre

- Utiliser une bibliothèque éprouvée (ex: `jsonwebtoken` pour Node.js, `PyJWT` pour Python)
- Mettre en place un mécanisme de rotation des refresh tokens
- Implémenter une blacklist Redis pour les tokens révoqués
- Créer des guides pour les développeurs sur l'utilisation des tokens
- Mettre en place des tests de sécurité (penetration testing)
- Documenter les flux d'authentification pour chaque type de client (web, mobile, API)

---

## Exemple 4 : Stratégie de Cache

### ADR-012 : Implémenter Redis pour le cache distribué

**Date** : 2024-04-20  
**Auteur** : Équipe Backend  
**Décideur** : Tech Lead  
**Tags** : performance, cache, redis, scalability

#### Statut
Accepté

#### Contexte

Notre application rencontre des problèmes de performance :

1. **Temps de réponse** : Les temps de réponse de certaines APIs dépassent 2 secondes
2. **Charge base de données** : La base de données est surchargée avec des requêtes répétitives
3. **Coûts** : Les coûts de la base de données augmentent avec la charge
4. **Scalabilité** : Difficulté à scaler la base de données (coûteux)

Analyses effectuées :
- 60% des requêtes sont des lectures de données qui changent rarement (catalogue produits, profils utilisateurs)
- Certaines requêtes complexes prennent 500ms-2s (rapports, analytics)
- Pic de trafic prévisible (ventes flash, événements)

Nous avons besoin d'une solution de cache qui :
- Réduise la charge sur la base de données
- Améliore les temps de réponse (< 200ms pour les données en cache)
- Soit distribuée (plusieurs instances d'application)
- Supporte l'invalidation intelligente
- Soit fiable et performante

#### Décision

Nous implémenterons **Redis** comme système de cache distribué.

**Architecture** :
- **Redis Cluster** : 3 nœuds en mode cluster pour haute disponibilité
- **Stratégie de cache** : Cache-aside pattern
- **TTL par type de données** :
  - Catalogue produits : 1 heure
  - Profils utilisateurs : 30 minutes
  - Données de session : 15 minutes
  - Données de configuration : 24 heures
- **Invalidation** : Invalidation explicite lors des mises à jour + TTL automatique
- **Fallback** : Si Redis est indisponible, requête directe à la DB (graceful degradation)

**Implémentation** :
- Bibliothèque : `redis-py` pour Python, `ioredis` pour Node.js
- Wrapper autour des accès DB pour transparent cache
- Monitoring des hit/miss rates
- Logging des performances

#### Alternatives considérées

##### 1. Memcached
**Avantages** :
- Plus simple que Redis
- Très performant pour le cache simple
- Moins de mémoire utilisée
- Bien établi

**Inconvénients** :
- Pas de persistance
- Pas de structures de données avancées
- Pas de réplication native
- Moins de fonctionnalités

**Pourquoi rejeté** : 
- Nous avons besoin de structures de données avancées (sets, sorted sets pour certains cas)
- Besoin de persistance optionnelle
- Redis offre plus de flexibilité pour l'avenir

##### 2. Cache applicatif (in-memory)
**Avantages** :
- Pas de dépendance externe
- Très rapide (pas de réseau)
- Simple à implémenter

**Inconvénients** :
- Pas distribué (chaque instance a son propre cache)
- Utilise la RAM de l'application
- Pas de partage entre instances
- Perte du cache au redémarrage

**Pourquoi rejeté** : 
- Nous avons plusieurs instances d'application (microservices)
- Besoin d'un cache partagé entre instances
- Limite la scalabilité

##### 3. Cache HTTP (CDN, Varnish)
**Avantages** :
- Très performant pour le cache HTTP
- Réduit la charge sur les serveurs
- Standard HTTP

**Inconvénients** :
- Seulement pour les réponses HTTP
- Pas pour le cache applicatif
- Moins flexible

**Pourquoi rejeté** : 
- Nous avons besoin de cache au niveau applicatif (pas seulement HTTP)
- Besoin de structures de données complexes
- Pas adapté à nos APIs

##### 4. Base de données avec cache intégré
**Avantages** :
- Pas de système supplémentaire à gérer
- Certaines DB ont un cache intégré

**Inconvénients** :
- Moins performant qu'un cache dédié
- Moins de contrôle
- Ne réduit pas vraiment la charge DB

**Pourquoi rejeté** : 
- Ne résout pas le problème de charge DB
- Performance inférieure à un cache dédié

#### Conséquences

##### Positives
- **Performance** : Réduction des temps de réponse de 80% pour les données en cache (2s → 50ms)
- **Charge DB** : Réduction de 60% de la charge sur la base de données
- **Coûts** : Réduction des coûts de base de données (moins de ressources nécessaires)
- **Scalabilité** : Meilleure scalabilité (cache distribué)
- **Disponibilité** : Redis Cluster offre haute disponibilité
- **Flexibilité** : Structures de données avancées disponibles si besoin

##### Négatives
- **Complexité** : Ajout d'un nouveau système à gérer (Redis)
- **Coûts infrastructure** : Coûts supplémentaires pour Redis (3 nœuds)
- **Consistance** : Risque d'incohérence cache/DB si invalidation mal gérée
- **Dépendance** : Nouvelle dépendance infrastructure (mais avec fallback)
- **Mémoire** : Redis utilise de la RAM (coûts)
- **Maintenance** : Surveillance et maintenance supplémentaires

##### À surveiller
- **Hit rate** : Taux de succès du cache (objectif : > 70%)
- **Performance** : Temps de réponse des requêtes en cache (objectif : < 100ms)
- **Mémoire Redis** : Utilisation de la mémoire (éviter l'éviction)
- **Disponibilité** : Uptime de Redis (objectif : 99.9%)
- **Incohérences** : Détecter les cas où le cache est incohérent avec la DB
- **Coûts** : Coûts infrastructure Redis vs économies sur la DB

#### Notes de mise en œuvre

- Mettre en place un monitoring complet (hit/miss rates, latence, mémoire)
- Créer des guidelines pour l'utilisation du cache (quand cacher, TTL, invalidation)
- Implémenter des tests de charge pour valider les gains de performance
- Documenter les patterns d'invalidation pour chaque type de données
- Mettre en place des alertes si Redis est indisponible
- Planifier la capacité (estimation de la mémoire nécessaire)

---

## Exemple 5 : Choix d'une Base de Données

### ADR-015 : Utiliser PostgreSQL comme base de données principale

**Date** : 2024-05-10  
**Auteur** : Équipe Backend  
**Décideur** : Tech Lead  
**Tags** : database, postgresql, infrastructure, data

#### Statut
Accepté

#### Contexte

Notre application utilise actuellement MySQL 5.7 comme base de données principale. MySQL 5.7 atteint sa fin de vie et nous devons migrer vers une nouvelle version.

Nos besoins actuels et futurs :
1. **Données relationnelles** : Beaucoup de relations complexes entre entités
2. **Transactions ACID** : Besoin de transactions fiables pour les paiements
3. **Requêtes complexes** : Beaucoup de JOINs et requêtes analytiques
4. **JSON** : Certaines données sont semi-structurées (métadonnées produits, préférences utilisateurs)
5. **Full-text search** : Recherche textuelle dans les descriptions produits
6. **Performance** : Besoin de bonnes performances pour les lectures et écritures
7. **Scalabilité** : Besoin de scaler verticalement et horizontalement (réplication)
8. **Extensions** : Possibilité d'ajouter des fonctionnalités (géolocalisation, etc.)

Contraintes :
- Budget limité (pas de solution propriétaire coûteuse)
- Équipe familière avec SQL
- Infrastructure cloud (AWS RDS)
- Besoin de migration depuis MySQL

#### Décision

Nous migrerons vers **PostgreSQL 15** comme base de données principale.

**Raisons principales** :
- Conformité SQL standard (meilleure que MySQL)
- Fonctionnalités avancées (JSON natif, arrays, full-text search amélioré)
- Performance supérieure pour les requêtes complexes
- Extensions puissantes (PostGIS pour géolocalisation, pg_trgm pour recherche)
- Licence open source permissive (PostgreSQL License)
- Excellente communauté et documentation
- Support natif pour les types de données avancés

**Plan de migration** :
1. **Phase 1** (Semaine 1-2) : Setup PostgreSQL en parallèle, tests de compatibilité
2. **Phase 2** (Semaine 3-4) : Migration des données avec `pgloader`, validation
3. **Phase 3** (Semaine 5-6) : Tests de régression complets, optimisation
4. **Phase 4** (Semaine 7-8) : Bascule progressive (blue-green deployment)

**Outils** :
- `pgloader` : Migration des données MySQL → PostgreSQL
- Scripts de validation pour comparer les données
- Tests de performance pour valider les améliorations

#### Alternatives considérées

##### 1. MySQL 8.0
**Avantages** :
- Migration plus simple (même famille)
- L'équipe connaît déjà MySQL
- Compatible avec notre code existant (syntaxe SQL similaire)
- Bonne performance
- Bien supporté par AWS RDS

**Inconvénients** :
- Moins de fonctionnalités avancées que PostgreSQL
- Performance inférieure pour les requêtes complexes avec beaucoup de JOINs
- JSON support moins mature
- Full-text search moins puissant
- Licence Oracle (préoccupations à long terme)

**Pourquoi rejeté** : 
- Nous avons besoin de fonctionnalités avancées (JSON natif, arrays)
- PostgreSQL offre de meilleures performances pour nos requêtes complexes
- Besoin d'extensions (géolocalisation) mieux supportées par PostgreSQL
- Préférence pour une licence plus permissive

##### 2. MariaDB 10.11
**Avantages** :
- Fork de MySQL, migration simple
- Open source (licence GPL)
- Compatible avec MySQL
- Bonne communauté

**Inconvénients** :
- Moins de fonctionnalités que PostgreSQL
- Performance similaire à MySQL (pas d'amélioration majeure)
- Écosystème moins large que PostgreSQL
- Moins d'extensions disponibles

**Pourquoi rejeté** : 
- Ne résout pas nos besoins de fonctionnalités avancées
- Pas d'amélioration significative par rapport à MySQL
- Écosystème moins riche que PostgreSQL

##### 3. MongoDB (NoSQL)
**Avantages** :
- Très flexible (schema-less)
- Excellente scalabilité horizontale
- Bon pour les données non-structurées
- Performance très bonne pour certaines requêtes

**Inconvénients** :
- Pas de transactions multi-documents fiables (à l'époque, maintenant disponible mais limité)
- Requiert un changement complet de modèle de données
- L'équipe n'a pas d'expérience avec NoSQL
- Pas adapté aux données fortement relationnelles
- Pas de JOINs (doit faire plusieurs requêtes)

**Pourquoi rejeté** : 
- Nos données sont fortement relationnelles (beaucoup de JOINs)
- Nous avons besoin de transactions ACID pour les paiements
- Le changement serait trop risqué et coûteux
- L'équipe devrait tout réapprendre

##### 4. Amazon Aurora PostgreSQL
**Avantages** :
- Gestion complète par AWS
- Scalabilité automatique
- Backups automatiques
- Haute disponibilité intégrée
- Performance très bonne

**Inconvénients** :
- Coût plus élevé que RDS standard (2-3x)
- Vendor lock-in avec AWS
- Pas nécessaire pour notre taille actuelle
- Moins de contrôle

**Pourquoi rejeté** : 
- Coûts trop élevés pour notre taille actuelle
- Nous commençons avec RDS PostgreSQL standard
- Nous réévaluerons Aurora quand nous aurons besoin de plus de scalabilité
- Préférence pour moins de vendor lock-in

#### Conséquences

##### Positives
- **Fonctionnalités** : Accès à des fonctionnalités avancées (JSON natif, arrays, full-text search amélioré, window functions)
- **Performance** : Meilleures performances pour les requêtes complexes avec JOINs (optimiseur de requêtes plus avancé)
- **Standard SQL** : Meilleure conformité aux standards SQL (portabilité)
- **Écosystème** : Large écosystème d'extensions (PostGIS pour géolocalisation, pg_trgm pour recherche floue, etc.)
- **Open source** : Licence PostgreSQL (plus permissive que MySQL Oracle)
- **Types de données** : Types de données avancés (UUID, arrays, JSONB, ranges)
- **Concurrence** : Meilleur système de verrous (MVCC plus efficace)

##### Négatives
- **Migration** : Migration complexe nécessitant des tests approfondis (2 mois estimés)
- **Formation** : L'équipe devra apprendre les spécificités de PostgreSQL (différences avec MySQL)
- **Coûts** : Coûts de migration et de formation
- **Risque** : Risque de bugs pendant la migration
- **Syntaxe** : Quelques différences de syntaxe SQL (LIMIT/OFFSET vs FETCH, etc.)
- **Outils** : Certains outils peuvent nécessiter des ajustements

##### À surveiller
- **Performance** : Surveiller les performances après migration (objectif : amélioration de 20-30%)
- **Compatibilité** : Vérifier la compatibilité de toutes les requêtes SQL
- **Formation** : S'assurer que l'équipe est formée aux différences MySQL/PostgreSQL
- **Backup** : Mettre en place une stratégie de backup robuste (WAL archiving)
- **Monitoring** : Configurer le monitoring spécifique à PostgreSQL (pg_stat_statements, etc.)
- **Migration** : Surveiller la migration des données (validation, performance)

#### Notes de mise en œuvre

- Créer un guide des différences MySQL/PostgreSQL pour l'équipe
- Utiliser `pgloader` pour la migration initiale des données
- Créer des scripts de validation pour comparer les données MySQL et PostgreSQL
- Mettre en place des tests de performance pour valider les améliorations
- Planifier une fenêtre de maintenance pour la bascule
- Avoir un plan de rollback prêt (garder MySQL en parallèle pendant 1 mois)
- Documenter les requêtes SQL spécifiques qui ont dû être adaptées

---

## Résumé des bonnes pratiques

À travers ces exemples, on peut identifier plusieurs bonnes pratiques :

1. **Contexte détaillé** : Expliquer clairement le problème et les contraintes
2. **Alternatives documentées** : Montrer que la décision a été réfléchie
3. **Conséquences réalistes** : Être honnête sur les avantages ET inconvénients
4. **Plan de mise en œuvre** : Inclure un plan concret pour l'implémentation
5. **Métriques** : Définir des objectifs mesurables à surveiller
6. **Notes pratiques** : Ajouter des notes pour faciliter l'implémentation

Ces exemples montrent comment documenter des décisions réelles avec suffisamment de détails pour être utiles, tout en restant concis et actionnables.
