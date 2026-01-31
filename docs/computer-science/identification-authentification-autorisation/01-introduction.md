# Introduction : identification, authentification et autorisation

## Pourquoi distinguer ces trois notions ?

Lorsqu’un utilisateur se connecte à une application ou accède à une ressource protégée, plusieurs questions se posent :

1. **Qui prétend accéder ?** → C’est l’**identification** : déclarer son identité (ex. login, email).
2. **Est-ce bien cette personne ?** → C’est l’**authentification** : prouver que l’on est bien qui l’on prétend être (ex. mot de passe, 2FA).
3. **A-t-elle le droit de faire cette action ?** → C’est l’**autorisation** : vérifier les droits (ex. rôles, permissions).

Si on mélange ces étapes, on obtient des designs flous (« l’authentification gère tout ») et des failles possibles (ex. autoriser une action sans avoir vérifié l’identité). Les distinguer permet de concevoir des contrôles d’accès clairs et maintenables.

## Les trois notions en une phrase

| Notion | Question posée | Exemple |
|--------|----------------|--------|
| **Identification** | Qui êtes-vous ? (déclaration) | Saisie du login ou de l’email |
| **Authentification** | Prouvez-le. (preuve) | Mot de passe, code 2FA, certificat |
| **Autorisation** | Avez-vous le droit ? (droits) | Rôle « admin », permission « supprimer » |

L’ordre logique est : d’abord **identifier** (prétendre être quelqu’un), puis **authentifier** (prouver que c’est vrai), enfin **autoriser** (vérifier si cette identité a le droit d’effectuer l’action demandée).

## Analogie : l’accès à un bâtiment sécurisé

Imaginez l’entrée d’un immeuble avec un gardien :

1. **Identification** : Vous dites « Je suis M. Dupont ». Vous **déclarez** votre identité (équivalent du login).
2. **Authentification** : Le gardien vérifie votre pièce d’identité ou vous demandez d’ouvrir la porte avec votre badge. Vous **prouvez** que vous êtes bien M. Dupont (équivalent du mot de passe ou du 2FA).
3. **Autorisation** : Une fois à l’intérieur, vous voulez entrer dans la salle des serveurs. Le gardien consulte la liste des personnes autorisées : « M. Dupont a le droit d’accéder au 3e étage, pas à la salle des serveurs. » C’est l’**autorisation** : même identité vérifiée, tous les droits ne sont pas accordés.

Sans identification, le gardien ne sait pas qui vérifier. Sans authentification, n’importe qui pourrait se faire passer pour M. Dupont. Sans autorisation, tout le monde pourrait aller partout une fois entré. Les trois étapes sont nécessaires et distinctes.

## Ordre dans un flux typique

Dans une application (web, API, etc.), le flux ressemble à ceci :

```
[Utilisateur]  →  Identification (login/email)
                      ↓
                 Authentification (mot de passe, 2FA, etc.)
                      ↓
                 Session ou token (identité reconnue)
                      ↓
                 Requête : « Je veux supprimer l’article X »
                      ↓
                 Autorisation (cet utilisateur a-t-il le droit ?)
                      ↓
                 Accès accordé ou refusé
```

L’identification et l’authentification interviennent en général **une fois** (à la connexion). L’autorisation intervient **à chaque requête** sur une ressource ou une action protégée : « Est-ce que l’utilisateur actuellement connecté a le droit de faire ça ? »

## Confusions courantes

### « Authentification » utilisé pour tout

En anglais, *authentication* est souvent employé pour « connexion » (identification + preuve). En français, on peut garder :
- **Identification** = déclarer qui on est
- **Authentification** = prouver qui on est
- **Autorisation** = vérifier les droits

Ainsi, « système d’authentification » peut désigner en pratique l’ensemble identification + authentification (login + mot de passe), mais il est utile de savoir que l’autorisation est une couche à part.

### Identification sans authentification

Si on accepte une identité sans preuve (ex. un simple champ « nom » non vérifié), n’importe qui peut se faire passer pour quelqu’un d’autre. L’identification doit être suivie d’une authentification pour être fiable.

### Autorisation sans identification ni authentification

Si on ne sait pas **qui** fait la requête (pas d’identification, pas d’authentification), on ne peut pas appliquer de règles d’autorisation cohérentes. L’autorisation repose sur une identité déjà établie et vérifiée.

## Résumé

- **Identification** : déclarer son identité (qui je prétends être).
- **Authentification** : prouver cette identité (mot de passe, 2FA, certificat, etc.).
- **Autorisation** : vérifier si cette identité a le droit d’effectuer l’action demandée (rôles, permissions).

Les trois notions sont distinctes et interviennent dans cet ordre. Les distinguer permet de concevoir des contrôles d’accès clairs et de mieux sécuriser les systèmes.

Dans la suite, nous détaillons chaque concept : [Concepts fondamentaux](./02-concepts-fondamentaux.md).
