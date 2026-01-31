# Concepts fondamentaux

Ce chapitre détaille les trois notions : identification, authentification et autorisation. Chacune répond à une question précise et s’appuie sur des mécanismes spécifiques.

## 1. Identification

### Définition

L’**identification** consiste à **déclarer son identité** : « Je suis l’utilisateur X. » Le système reçoit une **revendication d’identité** (claim) sans encore la vérifier.

### Question posée

**« Qui prétend accéder ? »**

### Mécanismes typiques

- **Identifiant (login)** : nom d’utilisateur unique (ex. `alice`, `user_42`)
- **Email** : adresse email utilisée comme identifiant (ex. `alice@example.com`)
- **Numéro de client** : identifiant métier (ex. `CLI-12345`)
- **Token ou certificat** : dans certains protocoles, l’identité est portée par un jeton (ex. JWT avec un champ `sub` pour le sujet)

L’identification fournit un **label** : « Cette requête concerne l’utilisateur dont l’identifiant est X. » Elle ne dit pas encore si la personne qui envoie la requête est bien X.

### Ce que l’identification n’est pas

- **Pas une preuve** : dire « je suis Alice » ne prouve pas que je suis Alice. C’est pourquoi l’authentification suit.
- **Pas un droit** : savoir qui prétend agir ne dit pas si cette personne a le droit d’effectuer l’action. C’est le rôle de l’autorisation.

### Exemple en pseudo-code

```java
// L'utilisateur envoie un identifiant (ex. formulaire de connexion)
String login = request.getParameter("login");  // "alice"
// À ce stade, on a une revendication d'identité, pas une preuve.
User claimedUser = userRepository.findByLogin(login);
if (claimedUser == null) {
    // Identifiant inconnu : on peut refuser sans aller jusqu'à l'authentification
    return "Identifiant inconnu";
}
// L'identification a « trouvé » un utilisateur ; il reste à prouver que
// la personne qui se connecte est bien cet utilisateur (authentification).
```

---

## 2. Authentification

### Définition

L’**authentification** consiste à **prouver** que l’on est bien l’entité que l’on prétend être. Le système vérifie la **preuve d’identité** fournie par l’utilisateur (ou par un système de confiance).

### Question posée

**« Prouvez que vous êtes bien qui vous prétendez être. »**

### Mécanismes typiques

- **Mot de passe** : secret partagé entre l’utilisateur et le système ; la comparaison (après hachage) prouve la connaissance du secret.
- **Code à usage unique (OTP)** : SMS, application type Google Authenticator ; prouve la possession d’un appareil ou d’un secret.
- **Authentification à deux facteurs (2FA / MFA)** : combinaison de deux preuves (ex. mot de passe + code OTP).
- **Certificat client** : le client présente un certificat numérique ; la clé privée prouve l’identité associée au certificat.
- **Clé API / token secret** : possession d’un secret (ex. clé API) considéré comme preuve d’identité pour les accès machine à machine.
- **Biométrie** : empreinte, reconnaissance faciale (souvent en complément d’un autre facteur).

En résumé : **quelque chose que vous savez** (mot de passe), **quelque chose que vous possédez** (téléphone, clé, certificat), ou **quelque chose que vous êtes** (biométrie).

### Ce que l’authentification n’est pas

- **Pas l’identification** : l’authentification suppose qu’une identité a déjà été déclarée (identification). On prouve ensuite que cette déclaration est vraie.
- **Pas l’autorisation** : prouver que je suis Alice ne dit pas si Alice a le droit de supprimer une ressource. Les droits sont gérés par l’autorisation.

### Exemple en pseudo-code

```java
// Après identification : on a un utilisateur revendiqué (claimedUser)
String password = request.getParameter("password");
boolean passwordValid = passwordHasher.verify(password, claimedUser.getPasswordHash());
if (!passwordValid) {
    return "Mot de passe incorrect";
}
// Authentification réussie : on crée une session ou un token
Session session = sessionService.createSession(claimedUser);
response.addCookie(new Cookie("SESSION_ID", session.getId()));
```

L’authentification valide la **revendication d’identité**. Une fois validée, le système peut considérer que les requêtes associées à cette session (ou ce token) émanent bien de l’utilisateur identifié.

---

## 3. Autorisation

### Définition

L’**autorisation** consiste à **vérifier si une identité (déjà identifiée et authentifiée) a le droit d’effectuer une action donnée** sur une ressource donnée.

### Question posée

**« Cette identité a-t-elle le droit de faire cette action ? »**

### Mécanismes typiques

- **Rôles** : l’utilisateur a un rôle (ex. `admin`, `éditeur`, `lecteur`). Chaque rôle est associé à un ensemble de droits.
- **Permissions** : droits explicites (ex. `article:delete`, `user:manage`). On vérifie si l’utilisateur (ou son rôle) possède la permission requise.
- **Contrôle d’accès par liste (ACL)** : pour une ressource donnée (ex. un fichier), une liste indique qui peut faire quoi (lecture, écriture, suppression).
- **Règles métier** : « L’utilisateur ne peut modifier que ses propres articles » ; la vérification dépend de l’identité et du propriétaire de la ressource.
- **Attributs (ABAC)** : décision basée sur des attributs (rôle, département, niveau de sécurité, etc.) plutôt que sur un rôle fixe.

L’autorisation intervient **après** que l’identité a été établie (identification + authentification). Elle répond à : « Étant donné que c’est bien Alice, Alice a-t-elle le droit de faire X ? »

### Ce que l’autorisation n’est pas

- **Pas l’identification ni l’authentification** : sans savoir qui fait la requête (identité vérifiée), on ne peut pas appliquer de règles d’autorisation cohérentes. L’autorisation s’appuie sur l’identité déjà reconnue.
- **Pas « tout ou rien »** : un même utilisateur peut être autorisé pour certaines actions et pas pour d’autres. L’autorisation est **par action et par ressource**.

### Exemple en pseudo-code

```java
// L'utilisateur est déjà authentifié ; son identité est dans la session
User currentUser = session.getUser();
String action = "article:delete";
Article article = articleRepository.findById(articleId);

// Autorisation : est-ce que currentUser a le droit de supprimer cet article ?
if (!authorizationService.can(currentUser, action, article)) {
    return Response.forbidden("Vous n'avez pas le droit de supprimer cet article");
}
articleRepository.delete(article);
```

L’autorisation peut combiner **rôle** (ex. « éditeur » peut supprimer) et **règle métier** (ex. « uniquement ses propres articles »).

---

## Synthèse : les trois concepts côte à côte

| Concept | Question | Entrée typique | Sortie typique |
|--------|----------|----------------|----------------|
| **Identification** | Qui prétend accéder ? | Login, email, identifiant | Utilisateur « revendiqué » (ou erreur « inconnu ») |
| **Authentification** | Prouvez-le. | Mot de passe, OTP, certificat | Session ou token (identité vérifiée) |
| **Autorisation** | A-t-il le droit ? | Identité + action + ressource | Autorisé ou refusé |

### Ordre et dépendances

1. **Identification** : on déclare une identité.
2. **Authentification** : on prouve cette identité → le système « connaît » l’utilisateur pour la suite de la session (ou du token).
3. **Autorisation** : à chaque action protégée, on vérifie si cette identité a le droit de l’effectuer.

Sans identification, pas d’authentification possible. Sans authentification, une identité déclarée n’est pas fiable, donc l’autorisation ne peut pas reposer dessus de façon sûre.

---

## Vocabulaire courant

- **IAA** (ou **AAA** en anglais) : Identification, Authentification, Autorisation.
- **AuthN** : abréviation courante pour *Authentication* (identification + preuve).
- **AuthZ** : abréviation courante pour *Authorization* (autorisation).
- **RBAC** : Role-Based Access Control — autorisation basée sur les rôles.
- **ABAC** : Attribute-Based Access Control — autorisation basée sur les attributs.

Dans la suite, nous voyons des [exemples concrets](./03-exemples-concrets.md) : flux de connexion, API, rôles et permissions.
