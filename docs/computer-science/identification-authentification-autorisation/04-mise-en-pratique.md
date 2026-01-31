# Mise en pratique

Ce guide vous aide à appliquer correctement l’identification, l’authentification et l’autorisation dans vos projets : bonnes pratiques, pièges à éviter et points d’attention pour l’intégration.

## Bonnes pratiques générales

### 1. Toujours enchaîner identification puis authentification

- **Identification** : récupérer la revendication d’identité (login, email).
- **Authentification** : vérifier cette identité par une preuve (mot de passe, 2FA, token).

Ne pas faire confiance à une simple déclaration d’identité sans preuve. Par exemple, ne pas autoriser une action sur la base d’un paramètre « userId » envoyé par le client sans avoir établi l’identité via une session ou un token vérifié.

### 2. Vérifier l’autorisation à chaque action protégée

L’autorisation ne doit pas être faite une seule fois à la connexion. À **chaque requête** sur une ressource ou une action protégée, vérifier : « L’utilisateur actuellement identifié et authentifié a-t-il le droit de faire cette action sur cette ressource ? »

Exemple à éviter : « L’utilisateur est connecté, donc on affiche le bouton Supprimer et on suppose qu’il a le droit. » Il faut aussi vérifier côté serveur avant d’exécuter la suppression.

### 3. Séparer les responsabilités dans le code

- **Identification** : recherche de l’utilisateur par identifiant (login, email).
- **Authentification** : vérification du mot de passe (ou autre preuve), création de session / token.
- **Autorisation** : service ou module qui répond à « `can(user, action, resource)` ».

Cela améliore la lisibilité et les tests : on peut tester l’autorisation avec des utilisateurs et des rôles fictifs sans refaire tout le flux de connexion.

### 4. Messages d’erreur distincts (sans tout révéler)

- **Identifiant inconnu** : « Identifiant ou mot de passe incorrect » (éviter de révéler qu’un email existe ou non).
- **Mot de passe incorrect** : même message générique pour ne pas aider un attaquant à deviner les comptes valides.
- **Non autorisé** : 403 Forbidden avec un message du type « Vous n’avez pas le droit d’effectuer cette action ».

En production, éviter de détailler les raisons (ex. « Vous n’avez pas le rôle admin ») si cela peut aider un attaquant.

---

## Pièges à éviter

### 1. Faire reposer l’autorisation sur des données envoyées par le client

**À éviter** : lire l’identité ou le rôle depuis un paramètre de requête ou un champ modifiable par le client (ex. `?userId=42`, body `{ "role": "admin" }`).

**Bon** : l’identité et les rôles viennent de la **session** ou du **token** vérifié côté serveur (signature, secret). Le client ne peut pas les falsifier.

### 2. Confondre « identifié » et « authentifié »

Un utilisateur peut être **identifié** (on a un login) sans être **authentifié** (pas encore prouvé par mot de passe ou 2FA). Ne pas accorder de droits tant que l’authentification n’a pas réussi. Par exemple, ne pas considérer comme « connecté » quelqu’un qui a seulement saisi son email.

### 3. Oublier l’autorisation sur certaines routes

Toute action sensible (suppression, modification, accès à des données personnelles, actions admin) doit être protégée par une **vérification d’autorisation** côté serveur, pas seulement par l’affichage ou le masquage de boutons côté client.

### 4. Mettre des informations sensibles dans un token non chiffré

Un JWT est en général **signé** mais **en base64** (lisible par quiconque possède le token). Ne pas y mettre de données sensibles (mot de passe, numéro de carte). Y mettre l’identité (id, roles) et des claims nécessaires à l’autorisation, et garder le token avec une expiration courte si possible.

### 5. Autorisation uniquement côté client

Masquer un bouton « Supprimer » pour les non-admins ne suffit pas. Un attaquant peut envoyer directement une requête `DELETE /api/articles/42`. L’**autorisation** doit être implémentée côté serveur (ou API) pour chaque action protégée.

---

## Intégration dans un projet

### Checklist pour une nouvelle fonctionnalité protégée

1. **Identification** : comment l’utilisateur est-il identifié ? (session, JWT, clé API)
2. **Authentification** : comment la preuve est-elle vérifiée ? (mot de passe, 2FA, validité du token)
3. **Autorisation** : pour cette action et cette ressource, quelle règle appliquer ? (rôle, permission, propriétaire de la ressource)
4. **Implémentation** : où placer la vérification d’autorisation ? (middleware, service, contrôleur)
5. **Tests** : tester le refus d’accès pour un utilisateur non autorisé (403) et l’accès pour un utilisateur autorisé.

### Où placer la logique d’autorisation ?

- **Middleware / filtre** : vérifier la présence d’une session ou d’un token valide (authentification), puis éventuellement un premier niveau d’autorisation (ex. « route réservée aux admins »).
- **Service d’autorisation** : centraliser les règles du type `can(user, action, resource)` pour les réutiliser dans les contrôleurs ou les services métier.
- **Contrôleur / endpoint** : appeler le service d’autorisation avant d’exécuter l’action (ex. avant de supprimer un article).

Exemple d’organisation :

```java
// Middleware : vérifie que l'utilisateur est authentifié
User user = authMiddleware.getCurrentUser(request);
if (user == null) return Response.unauthorized();

// Contrôleur : vérifie l'autorisation pour cette action
if (!authorizationService.can(user, "article:delete", article)) {
    return Response.forbidden();
}
articleService.delete(article);
```

### Ressources utiles

- **OWASP Authentication Cheat Sheet** : bonnes pratiques pour l’authentification.
- **OWASP Authorization Cheat Sheet** : bonnes pratiques pour l’autorisation.
- Documentation des frameworks utilisés : Spring Security, Passport.js, etc., pour les patterns d’intégration (filtres, guards, policies).

---

## Résumé

- **Identification** : déclarer son identité ; **authentification** : la prouver ; **autorisation** : vérifier les droits sur une action et une ressource.
- Enchaîner identification → authentification, puis vérifier l’autorisation **à chaque action** protégée.
- Ne jamais faire reposer l’identité ou les droits sur des données envoyées par le client sans vérification côté serveur.
- Séparer les responsabilités (identification, authentification, autorisation) dans le code et tester l’autorisation explicitement.

En appliquant ces principes, vous clarifiez le contrôle d’accès et vous renforcez la sécurité de vos applications.

---

Retour au [sommaire](./README.md) de la documentation Identification, authentification et autorisation.
