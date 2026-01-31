# Exemples concrets

Ce chapitre illustre l’identification, l’authentification et l’autorisation dans des cas concrets : flux de connexion web, API avec token, et contrôle d’accès par rôles (RBAC).

## 1. Flux de connexion (login) sur un site web

### Scénario

Un utilisateur ouvre la page de connexion, saisit son email et son mot de passe, puis clique sur « Se connecter ».

### Étapes IAA dans le flux

| Étape | Notion | Ce qui se passe |
|-------|--------|------------------|
| 1 | **Identification** | L’utilisateur envoie son email (ex. `alice@example.com`). Le serveur cherche un compte avec cet email. Si aucun compte n’existe, on peut répondre « Identifiant inconnu » sans demander de mot de passe. |
| 2 | **Authentification** | L’utilisateur envoie son mot de passe. Le serveur compare (après hachage) avec le hash stocké pour le compte trouvé. Si le mot de passe est incorrect, « Mot de passe incorrect ». Si correct, authentification réussie. |
| 3 | **Session** | Le serveur crée une session (ex. cookie `SESSION_ID` ou token JWT). Les requêtes suivantes portent ce cookie/token : le serveur sait **qui** est connecté sans redemander le mot de passe. |
| 4 | **Autorisation** (plus tard) | L’utilisateur clique sur « Supprimer l’article ». Le serveur lit l’identité depuis la session, puis vérifie : « Cet utilisateur a-t-il le droit de supprimer cet article ? » (rôle, propriétaire, etc.). Si oui → action ; si non → 403 Forbidden. |

### Exemple en pseudo-code (connexion)

```java
// POST /login
String email = request.getParameter("email");      // Identification
String password = request.getParameter("password");

User user = userRepository.findByEmail(email);
if (user == null) {
    return "Identifiant inconnu";  // Identification : aucun compte
}
if (!passwordHasher.verify(password, user.getPasswordHash())) {
    return "Mot de passe incorrect";  // Authentification échouée
}
// Authentification réussie
Session session = sessionService.createSession(user);
response.addCookie(new Cookie("SESSION_ID", session.getId()));
return redirect("/dashboard");
```

### Exemple en pseudo-code (action protégée + autorisation)

```java
// DELETE /articles/42
Session session = sessionService.getSession(request.getCookie("SESSION_ID"));
if (session == null) {
    return Response.unauthorized();  // Pas authentifié
}
User currentUser = session.getUser();  // Identité déjà établie
Article article = articleRepository.findById(42);
if (article == null) return Response.notFound();

// Autorisation : a-t-il le droit de supprimer cet article ?
if (!currentUser.hasRole("admin") && !article.getAuthorId().equals(currentUser.getId())) {
    return Response.forbidden("Vous ne pouvez pas supprimer cet article");
}
articleRepository.delete(article);
return Response.ok();
```

On voit clairement : **identification + authentification** à la connexion ; **autorisation** à chaque action sur une ressource protégée.

---

## 2. API REST avec token (JWT)

### Scénario

Une application cliente (front, mobile, autre service) appelle une API REST. L’utilisateur s’est déjà connecté ; le client envoie un **token JWT** dans l’en-tête `Authorization: Bearer <token>`.

### Rôle du token

- Le token JWT contient en général l’**identité** (ex. champ `sub` = subject, ou `userId`) et éventuellement des **rôles** ou **permissions**.
- Le fait de posséder un token valide (signé par le serveur) prouve que l’utilisateur a **réussi l’authentification** à un moment donné (login, OAuth, etc.). Le serveur ne redemande pas de mot de passe à chaque requête ; il **vérifie le token** (signature, expiration).
- Pour une requête donnée (ex. `DELETE /api/articles/42`), le serveur lit l’identité et les droits depuis le token (ou depuis la base), puis applique l’**autorisation** : cet utilisateur a-t-il le droit de supprimer l’article 42 ?

### Flux simplifié

```
[Client]  →  POST /login avec (email, password)
[Serveur] →  Identification + authentification
[Serveur] →  Génère un JWT (sub=userId, roles=[...], exp=...)
[Client]  →  Stocke le token
[Client]  →  DELETE /api/articles/42  avec  Authorization: Bearer <token>
[Serveur] →  Vérifie le token (signature, exp) → identité reconnue (authentification du token)
[Serveur] →  Autorisation : l’utilisateur « sub » a-t-il le droit de supprimer l’article 42 ?
[Serveur] →  200 OK ou 403 Forbidden
```

### Exemple en pseudo-code (vérification du token + autorisation)

```java
// Intercepteur / middleware pour les routes protégées
String authHeader = request.getHeader("Authorization");
if (authHeader == null || !authHeader.startsWith("Bearer ")) {
    return Response.unauthorized("Token manquant");
}
String token = authHeader.substring(7);
Claims claims = jwtService.verify(token);  // Signature + expiration
if (claims == null) {
    return Response.unauthorized("Token invalide ou expiré");
}
// Identité établie via le token (authentification « par token »)
String userId = claims.getSubject();
User currentUser = userRepository.findById(userId);

// Pour une action donnée, autorisation
if (!authorizationService.can(currentUser, "article:delete", articleId)) {
    return Response.forbidden();
}
// Suite du traitement...
```

Ici, **identification + authentification** sont « condensées » dans la vérification du JWT ; **autorisation** reste une étape distincte (vérification des droits pour l’action demandée).

---

## 3. Contrôle d’accès par rôles (RBAC)

### Scénario

Une application avec des rôles : `admin`, `éditeur`, `lecteur`. Chaque rôle a des permissions différentes. En plus, une règle métier : « Un éditeur ne peut modifier que ses propres articles. »

### Modèle simplifié

- **Utilisateur** : identité (id, login, etc.) + **rôles** (ex. `["éditeur"]`).
- **Rôle** : ensemble de **permissions** (ex. `article:read`, `article:write`, `article:delete`).
- **Règle métier** : « Peut modifier l’article X » si (permission `article:write` **et** (rôle admin **ou** auteur de l’article)).

### Identification et authentification

Comme dans les exemples précédents : l’utilisateur se connecte (identification + authentification). Une fois connecté, le système connaît son **identité** et ses **rôles** (stockés en base ou dans le token).

### Autorisation : vérifier rôle et règle métier

```java
// Vérification par rôle
boolean canDelete = currentUser.hasRole("admin") || currentUser.hasPermission("article:delete");

// Vérification avec règle métier : « uniquement ses propres articles »
boolean canEditArticle = currentUser.hasPermission("article:write")
    && (currentUser.hasRole("admin") || article.getAuthorId().equals(currentUser.getId()));

if (!canEditArticle) {
    return Response.forbidden("Vous ne pouvez pas modifier cet article");
}
```

On peut centraliser cela dans un service d’autorisation :

```java
// AuthorizationService
boolean can(User user, String action, Article article) {
    if ("article:delete".equals(action)) {
        return user.hasRole("admin") || user.hasPermission("article:delete");
    }
    if ("article:edit".equals(action)) {
        return user.hasPermission("article:write")
            && (user.hasRole("admin") || article.getAuthorId().equals(user.getId()));
    }
    return false;
}
```

L’**autorisation** s’appuie toujours sur une **identité déjà établie** (identification + authentification) et sur des **rôles / permissions / règles métier**.

---

## 4. Résumé des exemples

| Contexte | Identification | Authentification | Autorisation |
|----------|----------------|------------------|--------------|
| **Login web** | Email / login | Mot de passe (hash) | — (après connexion, à chaque action) |
| **API JWT** | Champ `sub` (ou équivalent) dans le token | Token signé et non expiré | Vérification des droits pour l’action demandée |
| **RBAC** | Identité de l’utilisateur connecté | Session ou token | Rôles, permissions, règles métier (ex. propriétaire de la ressource) |

Dans tous les cas : **identifier** puis **authentifier** pour savoir **qui** est l’utilisateur ; **autoriser** pour savoir **ce qu’il a le droit de faire** sur chaque ressource ou action.

La suite propose une [mise en pratique](./04-mise-en-pratique.md) : bonnes pratiques, pièges à éviter et intégration dans un projet.
