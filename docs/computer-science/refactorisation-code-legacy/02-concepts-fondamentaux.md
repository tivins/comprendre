# Concepts fondamentaux

Ce chapitre pose les bases : tests de caractérisation pour documenter le comportement actuel, points de couture (seams) pour isoler le legacy, et principe « encapsuler puis remplacer » pour avancer sans tout casser.

## 1. Tests de caractérisation

### Idée centrale

Un **test de caractérisation** ne valide pas une spécification idéale ; il enregistre le **comportement actuel** du code. On écrit un test qui appelle le code existant avec des entrées données et on fixe les assertions sur le résultat observé (ou sur les effets de bord : BDD, fichiers, appels). Si le code change et que le comportement voulu reste le même, le test reste vert ; si on refactore par erreur et qu’on change le comportement, le test échoue.

En pratique : pas de tests → on ne sait pas ce que fait vraiment le code. Les tests de caractérisation fournissent un **filet de sécurité** avant tout refactoring : ils ne disent pas si le comportement est « bon », mais ils signalent quand il change.

### Limite

Ces tests peuvent figer des bugs ou des comportements discutables. L’équipe doit décider : soit on les corrige (et on met à jour le test), soit on les documente (commentaire ou ticket) pour plus tard. Ne pas les ignorer sous prétexte que « ce n’est pas le bon comportement » — sans test, le refactoring reste dangereux.

---

## 2. Points de couture (seams)

### Définition

Un **seam** (point de couture) est un endroit du code où on peut changer le comportement **sans modifier le code appelant**. Typiquement : une dépendance injectée (constructeur, setter), un point d’extension (interface, callback), ou un remplacement par configuration (factory, feature flag). Le code legacy s’exécute « derrière » le seam ; on peut ensuite fournir une autre implémentation (nouveau module, mock en test) sans toucher aux appelants.

Exemple simple : au lieu d’instancier `new Mailer()` dans la classe, on reçoit un `Mailer` (ou une interface `MailerInterface`) au constructeur. L’appelant ne change pas ; on peut remplacer le `Mailer` par une implémentation de test ou une nouvelle implémentation réelle.

### Rôle dans le refactoring legacy

On **identifie** les dépendances qui bloquent les tests ou le remplacement (accès BDD direct, appel à une API, lecture de fichier). On **introduit un seam** (extraction d’interface, paramètre, factory) pour que cette dépendance soit fournie de l’extérieur. Ensuite on peut : mocker en test, ou brancher une nouvelle implémentation en prod sans toucher au reste du legacy.

### Compromis

Introduire un seam peut demander de modifier des signatures (constructeur, méthode). Si les appelants sont nombreux ou hors de votre contrôle (API publique), le coût monte. Parfois on commence par un seam « local » (une seule classe ou module) et on étend progressivement.

---

## 3. Encapsuler puis remplacer

### Principe

Au lieu de réécrire un gros bloc en une fois :

1. **Encapsuler** : faire en sorte que tout l’accès au legacy passe par une interface ou un point d’entrée unique (facade, service, adapter). Le reste de l’application ne parle qu’à cette interface.
2. **Tester** : ajouter des tests de caractérisation sur cette interface (comportement actuel).
3. **Remplacer** : implémenter une nouvelle version (plus propre, découplée) qui respecte la même interface ; basculer les appelants vers la nouvelle implémentation (config, feature flag, déploiement progressif). Une fois la migration terminée, supprimer l’ancien code.

Avantage : le système reste fonctionnel à chaque étape ; le risque est limité à la surface de l’interface.

### Quand c’est pertinent

- Module ou sous-système bien délimité (un service, un rapport, un import/export).
- Interface clairement définie (entrées/sorties identifiables). Si le legacy n’a pas d’interface nette, la première étape est d’en créer une (facade) sans changer le comportement interne.

### Limite

Si le legacy est imbriqué partout (appels directs dans des centaines de fichiers), l’encapsulation peut être coûteuse. On commence alors par les zones les plus critiques ou les plus souvent modifiées.

---

## 4. Dette technique et priorisation

Refactoriser du legacy consomme du temps. **Prioriser** : quelles parties bloquent les évolutions ou génèrent le plus de bugs ? Quelles parties sont stables et peu touchées ? Investir en priorité sur les premières ; accepter de laisser les secondes en l’état tant que le coût de les refactoriser dépasse le bénéfice attendu.

La « dette technique » n’est pas à rembourser intégralement d’un coup : c’est un arbitrage continu entre coût du changement et coût de ne pas changer.

---

## Synthèse rapide

| Concept | Rôle |
|--------|------|
| Tests de caractérisation | Documenter le comportement actuel pour sécuriser les refactorings |
| Seams | Permettre de changer une dépendance sans modifier les appelants |
| Encapsuler puis remplacer | Isoler le legacy derrière une interface, puis remplacer par étapes |

---

## Prochaines étapes

- **[Stratégies et approches](./03-strategies-et-approches.md)** : Strangler Fig, branche par abstraction, exécution parallèle, feature flags
- **[Exemples concrets](./04-exemples-concrets.md)** : extraction de service, tests sur code non testé, remplacement d’une dépendance
- **[Mise en pratique](./05-mise-en-pratique.md)** : quand refactoriser, pièges, intégration dans le workflow
