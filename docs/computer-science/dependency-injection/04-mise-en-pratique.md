# Mise en pratique de l'injection de dépendances

Ce guide vous aide à appliquer l'injection de dépendances dans vos projets : quand l'utiliser, comment commencer, pièges à éviter et intégration avec les principes SOLID et les frameworks.

## Quand utiliser l'injection de dépendances

### Utilisez la DI quand :

- Une classe a besoin d'un **service**, d'un **repository**, d'un **client** ou de tout collaborateur externe.
- Vous voulez **tester** cette classe en remplaçant les collaborateurs par des mocks ou des stubs.
- Vous voulez pouvoir **changer d'implémentation** (autre BDD, autre fournisseur, autre stratégie) sans modifier la classe.
- Vous visez une architecture **découplée** et le respect du **principe DIP** (SOLID).
- Plusieurs développeurs travaillent sur le même code et vous voulez des dépendances **explicites** (constructeur lisible).

### Réfléchir avant d'appliquer quand :

- **Scripts ou prototypes** très courts : l'overhead de créer des interfaces et d'assembler les objets peut être disproportionné.
- **Objets de données** (DTO, entités) : ils n'ont en général pas besoin de dépendances injectées ; ce sont des structures de données.
- **Point d'entrée unique** (main, bootstrap) : c'est justement l'endroit où on **fait** l'assemblage et l'injection ; on n'injecte pas dans le main, on y construit le graphe d'objets.

### À éviter :

- **Injecter des primitives ou des configs brutes** partout : préférer un objet de configuration injecté une fois (ex. `AppConfig`) plutôt que dix paramètres String/int dans le constructeur.
- **Tout mettre en interface** sans raison : si une classe n'a qu'une seule implémentation et n'est pas mockée, une interface peut être superflue au début ; on peut l'ajouter au besoin.
- **Service Locator** à la place de la DI : la classe ne doit pas aller chercher ses dépendances dans un registre global ; elle doit les recevoir.

---

## Comment commencer

### Approche progressive

1. **Identifier les dépendances cachées** : repérer les `new ConcreteClass()` ou les appels à un Service Locator / singleton à l'intérieur des classes métier.
2. **Extraire des interfaces** (si besoin) : pour chaque dépendance que vous voulez pouvoir remplacer (tests, autre implémentation), définir une interface et faire implémenter la classe concrète.
3. **Passer les dépendances par le constructeur** : modifier le constructeur pour recevoir les interfaces (ou les classes concrètes si vous n'avez pas encore d'interface).
4. **Centraliser l'assemblage** : créer les objets concrets à un seul endroit (composition root ou conteneur IoC) et y construire les services en passant les dépendances.
5. **Tester** : écrire des tests unitaires qui injectent des mocks ; vérifier que les tests sont plus simples et que le code métier reste inchangé.

### Premier refactoring type

**Avant** (couplage fort) :

```java
class OrderService {
    private final OrderRepository orderRepository = new JdbcOrderRepository();
    // ...
}
```

**Après** (injection par constructeur) :

```java
class OrderService {
    private final OrderRepository orderRepository;

    public OrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }
    // ...
}

// À la racine de l'application
OrderRepository repo = new JdbcOrderRepository(dataSource);
OrderService orderService = new OrderService(repo);
```

Une fois cette étape en place, vous pouvez introduire une interface `OrderRepository` et faire implémenter `JdbcOrderRepository` si vous voulez pouvoir mocker ou changer d'implémentation.

---

## Pièges à éviter

### 1. Trop de dépendances dans une même classe

**Symptôme** : Un constructeur avec 6, 8 ou 10 paramètres.

**Risque** : La classe fait probablement trop de choses (violation du SRP) ; le code devient difficile à lire et à maintenir.

**Pistes** :
- Regrouper des dépendances liées dans un objet (ex. `OrderDependencies` contenant repository + notification + stock) seulement si cela a du sens métier.
- Sinon, repenser la responsabilité de la classe : la diviser en plusieurs services plus focalisés.

### 2. Dépendances circulaires

**Symptôme** : A dépend de B, B dépend de C, C dépend de A. Le conteneur IoC ou votre composition root ne peut pas construire les objets.

**Pistes** :
- Introduire une interface ou un événement pour casser le cycle (ex. C ne dépend pas de A mais d'une interface que A implémente).
- Regrouper les responsabilités : si A, B et C sont toujours utilisés ensemble, peut-être qu’un seul composant doit les contenir.
- Revoir le design : les dépendances circulaires sont souvent un signal que les frontières entre modules sont mal placées.

### 3. Injection de valeurs de configuration partout

**Symptôme** : Chaque service reçoit en constructeur l’URL de la BDD, la clé API, le port SMTP, etc.

**Piste** : Créer un objet (ou une interface) de configuration injecté une fois, par exemple `SmtpConfig`, `DatabaseConfig`, et les injecter dans les services qui en ont besoin. Les services reçoivent alors une config déjà regroupée et typée.

### 4. Oublier de mettre à jour la composition root

**Symptôme** : Vous ajoutez une nouvelle dépendance dans le constructeur d’un service mais vous oubliez de l’instancier et de la passer là où le service est créé. Erreur à l’exécution (NullPointerException ou erreur du conteneur).

**Piste** : Garder l’assemblage (composition root ou configuration IoC) à jour à chaque changement de constructeur ; éventuellement ajouter des tests d’intégration qui démarrent l’application ou le graphe d’objets pour détecter les oublis.

### 5. Utiliser le Service Locator en pensant faire de la DI

**Symptôme** : La classe appelle `ServiceLocator.get(OrderRepository.class)` dans le constructeur ou dans une méthode.

**Problème** : Les dépendances ne sont pas explicites (on ne les voit pas dans la signature du constructeur) et le code reste couplé au Service Locator. Ce n’est pas de l’injection de dépendances.

**Piste** : Remplacer par une vraie injection : le constructeur reçoit `OrderRepository` ; c’est l’appelant (ou le conteneur) qui obtient l’instance et la passe.

---

## Intégration avec SOLID et les frameworks

### Lien avec le principe DIP (SOLID)

L’injection de dépendances est la **technique** qui permet d’appliquer le **Dependency Inversion Principle** :

- Les modules de haut niveau (services, orchestrateurs) ne doivent pas dépendre des modules de bas niveau (implémentations concrètes).
- Les deux doivent dépendre d’abstractions (interfaces).

En injectant des interfaces (ou des abstractions) plutôt qu’en instanciant des classes concrètes, vous respectez le DIP. La DI est donc un moyen privilégié pour le mettre en œuvre.

### Utilisation avec un conteneur IoC (Spring, Jakarta CDI, etc.)

Dans les frameworks modernes :

- Vous déclarez les beans (classes ou interfaces) et leurs dépendances (annotations ou configuration).
- Le conteneur construit le graphe d’objets et injecte les dépendances (souvent par constructeur ou par setter).

**Bonnes pratiques** :
- Privilégier l’injection par **constructeur** (recommandé par Spring et beaucoup d’équipes).
- Dépendre d’**interfaces** pour les services métier et techniques (repository, notification, etc.) afin de faciliter les mocks et les changements d’implémentation.
- Garder la **composition root** logique dans un module de configuration ou un package dédié, plutôt que d’éparpiller la création d’objets dans tout le code.

### Sans framework : composition root manuelle

Dans un projet sans framework IoC :

- Choisir un **point d’entrée** (classe `Main`, `Application`, ou module « bootstrap ») où tout l’assemblage est fait.
- Y créer les implémentations concrètes (repositories, services, clients).
- Y construire les services métier en passant les dépendances par le constructeur.
- Exposer les services nécessaires (par exemple un `OrderService`) au reste de l’application (HTTP, CLI, etc.).

C’est de l’injection de dépendances « manuelle » : même principe (découplage, testabilité), sans conteneur.

---

## Checklist rapide

- [ ] Les classes métier ne font pas `new` sur des services/repositories ; elles les reçoivent par le constructeur (ou, plus rarement, par setter).
- [ ] Les dépendances nécessaires sont **visibles** dans la signature du constructeur.
- [ ] Il existe un **seul endroit** (composition root ou conteneur) où les implémentations concrètes sont créées et injectées.
- [ ] Les tests unitaires **injectent des mocks** au lieu d’utiliser de vrais services ou bases de données.
- [ ] Aucune classe ne dépend d’un **Service Locator** ou d’un singleton global pour obtenir ses collaborateurs.
- [ ] Les dépendances **optionnelles** sont gérées explicitement (setter, ou objet « no-op » injecté) plutôt que par des singletons cachés.

---

## Récapitulatif

- **Quand** : Dès qu’une classe a des collaborateurs (repository, service, client) que vous voulez pouvoir remplacer (tests, autre implémentation).
- **Comment** : Injection par **constructeur** en priorité ; setter pour dépendances optionnelles ; assemblage centralisé (composition root ou conteneur IoC).
- **Pièges** : Trop de dépendances, dépendances circulaires, config éparpillée, oubli de la composition root, confusion avec le Service Locator.
- **Intégration** : La DI est le moyen privilégié pour appliquer le **DIP** (SOLID) ; elle s’intègre avec les conteneurs IoC (Spring, CDI) ou peut se faire manuellement dans une composition root.

En appliquant ces points, vous obtiendrez un code plus découplé, plus testable et plus maintenable, sans complexifier inutilement les parties simples de votre projet.
