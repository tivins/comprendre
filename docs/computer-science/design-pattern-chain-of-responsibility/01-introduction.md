# Introduction au pattern Chaîne de responsabilité

## Qu'est-ce que la Chaîne de responsabilité ?

La **Chaîne de responsabilité** (Chain of Responsibility) est un pattern comportemental qui permet de **faire circuler une requête** le long d'une chaîne d'objets (handlers). Chaque objet peut soit traiter la requête, soit la transmettre au successeur. Ainsi, l'émetteur n'a pas besoin de connaître tous les traitants possibles : il envoie la requête au premier maillon, et c'est la chaîne qui décide qui traite.

### Intention (GoF)

> Éviter le couplage entre l'émetteur d'une requête et ses récepteurs en donnant à plusieurs objets la possibilité de traiter la requête. Chaînez les objets récepteurs et passez la requête le long de la chaîne jusqu'à ce qu'un objet la traite.

En résumé : **découpler l'émetteur des traitants** et laisser la requête "remonter" ou "descendre" la chaîne jusqu'à ce qu'un handler la prenne en charge.

## Le problème : pourquoi pas un gros switch ?

### Exemple problématique

Imaginons un système qui doit **traiter différents types de requêtes** : validation d'un formulaire, gestion d'erreurs HTTP, approbation de demandes selon le montant, etc.

**Avec des conditions en cascade** :

```java
void processRequest(Request request) {
    if (request.getType().equals("VALIDATION")) {
        validate(request);
    } else if (request.getType().equals("ERROR_404")) {
        handle404(request);
    } else if (request.getType().equals("ERROR_401")) {
        handle401(request);
    } else if (request.getType().equals("ERROR_500")) {
        handle500(request);
    } else if (request.getType().equals("APPROVAL_LOW")) {
        approveLow(request);
    } else if (request.getType().equals("APPROVAL_HIGH")) {
        approveHigh(request);
    }
    // ... des dizaines de cas
}
```

**Problèmes** :
- **Couplage fort** : l'appelant doit connaître tous les types de requêtes et tous les traitants.
- **Code rigide** : ajouter un nouveau type oblige à modifier ce bloc (violation du principe Open/Closed).
- **Responsabilité unique bafouée** : une seule méthode concentre toute la logique de routage.
- **Testabilité réduite** : difficile de tester un traitement isolé sans exécuter tout le switch.

### Ce qu'apporte la Chaîne de responsabilité

Avec la Chaîne de responsabilité, on construit une **chaîne de handlers**. Chaque handler connaît éventuellement un "successeur". Il reçoit la requête : s'il peut la traiter, il le fait ; sinon, il la passe au successeur.

```java
// Le client ne connaît que le premier maillon
Handler chain = new ValidationHandler();
chain.setNext(new Error404Handler())
     .setNext(new Error401Handler())
     .setNext(new Error500Handler());

// Une seule entrée : la requête circule toute seule
chain.handle(request);
```

**Avantages** :
- **Découplage** : l'émetteur ne connaît que le premier handler ; les traitants sont ajoutés/retirés sans toucher au client.
- **Ouvert à l'extension** : nouveau type de requête → nouveau handler, sans modifier les autres.
- **Responsabilités séparées** : chaque handler gère un cas précis.
- **Ordre configurable** : on peut réorganiser la chaîne (ordre de validation, priorité des erreurs, etc.).

## Analogie du quotidien

Pensez à une **réclamation client** dans une entreprise :
- La réclamation arrive au **standard** (premier maillon).
- Si le standard peut la traiter (ex. simple question de facturation), il le fait et s'arrête.
- Sinon, il transmet au **service support**.
- Le support traite s'il peut (ex. bug logiciel), sinon il transmet au **technique**.
- Le technique traite ou transmet au **responsable**, etc.

Personne n'a besoin de connaître toute l'organisation : chacun connaît seulement son successeur. La réclamation "circule" jusqu'à ce qu'elle soit prise en charge.

La Chaîne de responsabilité fonctionne de la même façon : une requête circule de handler en handler jusqu'à ce qu'un maillon la traite (ou jusqu'à la fin de la chaîne).

## Relation avec les principes SOLID

### Principe Open/Closed (OCP)

La Chaîne de responsabilité illustre l'OCP :
- **Ouvert à l'extension** : on ajoute un nouveau handler sans modifier les handlers existants ni le client.
- **Fermé à la modification** : le code des handlers en place et du client reste inchangé.

### Principe de responsabilité unique (SRP)

Chaque handler a une seule raison de changer : le type de requête (ou la règle) qu'il gère. La logique de routage est déléguée à la structure de la chaîne, pas à une grosse méthode centrale.

### Principe d'inversion des dépendances (DIP)

Le client dépend d'une abstraction (Handler), pas des implémentations concrètes. La chaîne est construite en utilisant des références vers l'interface Handler.

## Quand le pattern est utile

- **Plusieurs objets peuvent traiter une requête** et vous ne voulez pas que l'émetteur choisisse explicitement lequel.
- Vous voulez **spécifier les handlers dynamiquement** (ordre, ajout/suppression de maillons).
- Vous voulez **éviter le couplage** entre l'émetteur et la liste des récepteurs.
- Vous avez des **pipelines** : validation en plusieurs étapes, gestion d'erreurs par type, niveaux d'approbation, middleware (ex. requêtes HTTP).

## Ce que la Chaîne de responsabilité n'est pas

- **Pas un Decorator** : en Chaîne de responsabilité, un seul handler traite la requête (en général) et peut arrêter la propagation ; le Decorator enveloppe un composant et enrichit systématiquement le comportement en déléguant toujours.
- **Pas un simple liste de traitements** : chaque maillon décide s'il traite ou passe au suivant ; la requête ne passe pas nécessairement par tous les maillons.
- **Pas une file d'attente** : l'ordre est une chaîne de responsabilité (successeur fixe), pas une file où plusieurs consommateurs tirent des messages.

## Prochaines étapes

La suite de la documentation détaille :
- **[Structure et principe](./02-structure-et-principe.md)** : Handler abstrait, handlers concrets, successeur, passage de la requête
- **[Exemples concrets](./03-exemples-concrets.md)** : validation, erreurs HTTP, pipeline, niveaux d'approbation
- **[Mise en pratique](./04-mise-en-pratique.md)** : quand l'utiliser, alternatives, pièges à éviter

---

**Rappel** : La Chaîne de responsabilité est une solution pour découpler l'émetteur des traitants et faire circuler une requête jusqu'à prise en charge. Utilisez-la lorsque plusieurs objets peuvent traiter la requête et que vous voulez garder l'ordre et la liste des traitants flexibles.
