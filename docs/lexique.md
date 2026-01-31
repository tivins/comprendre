# Lexique — Conception et architecture

Définitions courtes et concrètes des termes récurrents dans la doc. Pour aller plus loin, chaque entrée pointe vers les sujets concernés.

---

## Couplage

**Définition** : degré de dépendance d’un composant (classe, module, service) envers d’autres. Un **couplage fort** signifie qu’un changement dans A impose souvent des changements dans B : types concrets, appels directs, connaissances internes partagées.

**En pratique** : une classe qui instancie elle-même un `SmtpEmailService` ou qui appelle une API HTTP en dur est fortement couplée à ces implémentations. Conséquences : tests difficiles (impossible de remplacer le mailer ou l’API), évolution coûteuse (changer de fournisseur = modifier la classe).

→ Voir [Injection de dépendances](computer-science/dependency-injection/README.md), [SOLID](computer-science/solid/README.md).

---

## Découplage

**Définition** : réduire le couplage entre composants pour qu’ils évoluent de façon plus indépendante. Concrètement : dépendre d’abstractions (interfaces, contrats) plutôt que d’implémentations, et recevoir les dépendances de l’extérieur au lieu de les créer.

**En pratique** : au lieu que `OrderService` crée un `SmtpEmailService`, il reçoit un `EmailService` (interface) ; les tests injectent un faux, la prod injecte l’implémentation réelle. Le découplage n’est pas une fin en soi : une couche d’abstractions mal pensée peut recréer du couplage caché (par exemple via un service locator global).

→ Voir [Injection de dépendances](computer-science/dependency-injection/README.md), [SOLID — Dependency Inversion](computer-science/solid/06-dependency-inversion.md).

---

## Responsabilité

**Définition** : ce qu’un composant est censé faire — le « rôle » ou la raison d’être qui justifie son existence. Une **responsabilité unique** (SRP) signifie une seule raison principale de changer : si une classe gère à la fois la persistance et l’envoi d’emails, deux axes de changement (BDD, politique d’envoi) la touchent.

**En pratique** : séparer « enregistrer une commande » et « notifier le client » en deux composants distincts améliore la testabilité et la clarté. La frontière reste un choix de conception : trop de micro-classes peut nuire à la lisibilité ; trop de responsabilités mélangées augmente la fragilité.

→ Voir [SOLID — Single Responsibility](computer-science/solid/02-single-responsibility.md), [Design patterns](computer-science/design-pattern/README.md).

---

## Dépendance

**Définition** : tout objet ou service dont un composant a besoin pour fonctionner (repository, client HTTP, logger, etc.). La **dépendance** désigne à la fois la relation (« A dépend de B ») et l’objet injecté (« B est une dépendance de A »).

**En pratique** : déclarer les dépendances dans le constructeur (injection par constructeur) rend le besoin explicite et facilite les tests en passant des doublures. Une dépendance cachée (création interne, singleton, service locator) complique la compréhension et le remplacement en test.

→ Voir [Injection de dépendances](computer-science/dependency-injection/README.md).

---

## Cohésion

**Définition** : degré auquel les éléments d’un composant (méthodes, champs) sont liés par une même responsabilité ou un même axe de changement. Une **forte cohésion** : tout ce qui est dans la classe sert une même idée ; une faible cohésion : la classe mélange des rôles sans lien fort.

**En pratique** : une classe « fourre-tout » (validation + envoi d’email + log + persistance) a une faible cohésion et souvent plusieurs responsabilités. Viser une cohésion élevée va de pair avec le SRP : une classe cohérente a en général une raison de changer bien identifiée.

→ Voir [SOLID — Single Responsibility](computer-science/solid/02-single-responsibility.md).

---

## Interface / Contrat

**Définition** : en conception, une **interface** (ou contrat) décrit ce qu’un composant expose — méthodes, paramètres, comportement attendu — sans imposer l’implémentation. Le code qui utilise l’interface ne dépend pas du détail interne.

**En pratique** : en POO, une interface (ou une classe abstraite) définit le contrat ; plusieurs implémentations peuvent coexister (ex. `EmailService` avec `SmtpEmailService`, `SendGridEmailService`, `FakeEmailService` pour les tests). En architecture, un contrat peut être un message, un API REST ou un événement : l’important est que l’appelant dépende du contrat, pas de l’implémentation de l’autre côté.

→ Voir [SOLID — Interface Segregation, Dependency Inversion](computer-science/solid/README.md), [Bus applicatif](computer-science/bus-applicatif/README.md).

---

*Lexique à compléter au fil des sujets. Proposition : **abstraction**, **inversion de dépendances**, **événement**, **commande**.*
