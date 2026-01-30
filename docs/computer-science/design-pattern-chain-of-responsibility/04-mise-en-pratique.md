# Mise en pratique de la Chaîne de responsabilité

Ce guide aide à décider quand utiliser la Chaîne de responsabilité, comment l'appliquer pas à pas, et quelles alternatives ou pièges éviter.

## Quand utiliser la Chaîne de responsabilité ?

### Utilisez la Chaîne de responsabilité quand :

1. **Plusieurs objets peuvent traiter une requête** : Vous ne voulez pas que l'émetteur choisisse explicitement lequel (éviter un gros `switch` ou une longue série de `if/else`).
2. **Handlers spécifiés dynamiquement** : Vous voulez pouvoir ajouter, retirer ou réordonner les traitants sans modifier le client ni les autres handlers.
3. **Découplage émetteur / récepteurs** : L'émetteur ne doit pas dépendre de la liste concrète des traitants, seulement du premier maillon de la chaîne.
4. **Pipeline ou ordre de priorité** : L'ordre des handlers a du sens (ex. validation "requis" avant "format", erreur 404 avant erreur 500 en fallback).
5. **Un seul handler traite (ou tous jusqu'à échec)** : Soit un seul maillon prend la requête et s'arrête, soit plusieurs maillons s'enchaînent jusqu'à la première erreur (ex. validation).

### Exemples de bons cas d'usage

- **Validation** : formulaire, champs (requis, longueur, format) ; premier échec arrête la chaîne.
- **Gestion d'erreurs** : codes HTTP, exceptions ; un handler par type d'erreur.
- **Niveaux d'approbation** : congés, remboursements, montants ; premier niveau compétent traite.
- **Middleware** : requêtes HTTP (auth, logging, cache) ; la requête traverse une chaîne de traitements.
- **Événements / logs** : plusieurs récepteurs possibles ; la requête circule jusqu'à prise en charge.

## Quand éviter la Chaîne de responsabilité ?

### Évitez la Chaîne de responsabilité quand :

1. **Un seul traitant** : Un seul objet doit toujours traiter la requête → un simple appel de méthode suffit.
2. **Ordre fixe et connu du client** : Si le client doit impérativement savoir qui traite et dans quel ordre, le découplage n'apporte pas grand-chose ; une stratégie ou un dispatcher explicite peut suffire.
3. **Tous les handlers doivent toujours s'exécuter** : Si chaque maillon doit systématiquement faire un travail (enrichissement, log), le pattern **Decorator** ou un **pipeline** où la requête traverse tous les maillons est plus adapté.
4. **Requête non propagée** : Si la "requête" est modifiée de façon incompatible entre maillons (changement de type, de contrat), le pattern devient difficile à maintenir ; envisager des étapes explicites (pipeline avec types distincts).
5. **Chaîne trop longue ou trop complexe** : Beaucoup de maillons et des dépendances entre eux rendent le flux difficile à suivre ; envisager une machine à états ou un orchestrateur.

### Alternatives à considérer

| Besoin | Alternative possible |
|--------|------------------------|
| Un seul traitant, connu à l'avance | Appel direct de méthode |
| Tous les maillons doivent s'exécuter (enrichissement) | **Decorator** ou pipeline explicite |
| Choisir un algorithme parmi plusieurs | **Strategy** |
| Déclencher plusieurs récepteurs (notification) | **Observer** |
| File d'attente avec plusieurs consommateurs | File + workers (pas une chaîne de successeur unique) |
| Routage complexe (règles métier nombreuses) | Table de routage, règles, ou moteur de règles |

## Étapes pour appliquer le pattern

### Étape 1 : Définir la requête ou le contexte

- Quelles données circulent le long de la chaîne ? (valeur à valider, code d'erreur, demande d'approbation, etc.)
- Sous quelle forme ? (objet Request, paramètres, contexte partagé)

Définir une **classe (ou structure) de requête/contexte** avec les champs nécessaires.

```java
class Request {
    private String type;
    private int code;
    private String message;
    // getters, setters, constructeur
}
```

### Étape 2 : Créer le Handler abstrait

- Une classe abstraite (ou interface avec implémentation de base) avec une référence vers le **successeur** (next).
- Une méthode `setNext(Handler next)` qui permet de chaîner (et qui peut retourner `next` pour un chaînage fluide).
- Une méthode `handle(...)` qui décide si le handler traite la requête ou la transmet au successeur.

```java
abstract class Handler {
    protected Handler next;
    public Handler setNext(Handler next) {
        this.next = next;
        return next;
    }
    public abstract boolean handle(Request request);
}
```

### Étape 3 : Implémenter les Handlers concrets

- Une classe par type de traitement (validation requise, erreur 404, approbation manager, etc.).
- Chaque handler implémente la logique "puis-je traiter ?" et "comment je traite ?".
- S'il ne traite pas, il appelle `next.handle(request)` si `next != null`.

```java
class ConcreteHandlerA extends Handler {
    public boolean handle(Request request) {
        if (canHandle(request)) {
            doHandle(request);
            return true;
        }
        if (next != null) {
            return next.handle(request);
        }
        return false;
    }
    private boolean canHandle(Request request) { /* ... */ }
    private void doHandle(Request request) { /* ... */ }
}
```

### Étape 4 : Construire la chaîne et l'utiliser

- Le client construit la chaîne en enchaînant les `setNext`.
- Le client n'appelle que le premier handler ; la requête circule toute seule.

```java
Handler chain = new HandlerA();
chain.setNext(new HandlerB()).setNext(new HandlerC());

chain.handle(request);
```

## Pièges à éviter

### 1. Oublier de définir un "dernier" handler

Si aucune condition n'est jamais remplie, la requête arrive en fin de chaîne. Prévoir soit un **handler par défaut** qui traite toujours, soit un comportement clair (retour `false`, exception, log). Sinon, la requête peut être "avalée" sans retour explicite.

### 2. Ordre des maillons non documenté

L'ordre de la chaîne détermine la priorité ou l'ordre de traitement. Si l'ordre est critique (ex. validation "requis" avant "format"), le documenter ou le figer dans une factory pour éviter des chaînes incorrectes.

### 3. Requête modifiée de façon incohérente

Si un handler modifie la requête (contexte mutable), les handlers suivants voient la version modifiée. C'est parfois voulu (pipeline d'enrichissement) ; sinon, éviter les mutations partagées ou utiliser des copies pour garder un comportement prévisible.

### 4. Confusion Chaîne de responsabilité / Decorator

- **Chaîne de responsabilité** : un handler traite **ou** passe au suivant ; la requête ne passe pas nécessairement par tous les maillons.
- **Decorator** : chaque décorateur **enveloppe** un composant et **délègue toujours** en ajoutant un comportement ; le client voit une seule interface, tous les décorateurs participent.

Ne pas utiliser la Chaîne de responsabilité pour "enrichir" systématiquement une requête à chaque maillon ; dans ce cas, le Decorator ou un pipeline explicite est plus adapté.

### 5. Chaîne trop longue ou trop couplée

Au-delà de quelques maillons, le flux devient difficile à déboguer et à tester. Envisager des regroupements (sous-chaînes), des factories qui construisent des chaînes prédéfinies, ou un autre modèle (orchestrateur, machine à états).

## Checklist rapide

Avant d'introduire une Chaîne de responsabilité, vérifier :

- [ ] **Plusieurs objets** peuvent-ils traiter la requête ?
- [ ] Voulez-vous **découpler** l'émetteur de la liste des traitants ?
- [ ] L'**ordre** des handlers a-t-il du sens (priorité, pipeline) ?
- [ ] Un seul handler traite à la fois (ou tous jusqu'à échec) → la sémantique correspond-elle au pattern ?

Si la majorité des réponses est oui, la Chaîne de responsabilité est un bon candidat. Sinon, envisager un appel direct, Strategy, Observer ou Decorator.

## Aller plus loin

- **[Design Patterns (patterns comportementaux)](../design-pattern/04-patterns-comportementaux.md)** : Comparaison avec Observer, Strategy, Command, State.
- **[Design pattern Decorator](../design-pattern-decorator/01-introduction.md)** : Différence avec la Chaîne de responsabilité (enrichissement vs traitement conditionnel).
- **[Exemples concrets](./03-exemples-concrets.md)** : Revoir validation, erreurs HTTP, approbation, pipeline.

---

**Rappel** : La Chaîne de responsabilité sert à découpler l'émetteur des traitants et à faire circuler une requête jusqu'à prise en charge. Utilisez-la lorsque plusieurs objets peuvent traiter la requête et que l'ordre ou la liste des traitants doit rester flexible ; sinon, privilégiez la solution la plus simple.
