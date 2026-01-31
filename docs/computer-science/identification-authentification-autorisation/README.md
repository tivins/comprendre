# Identification, authentification et autorisation

## Introduction

En sécurité informatique, **identification**, **authentification** et **autorisation** sont trois notions distinctes qui interviennent lorsqu’un utilisateur accède à un système ou à une ressource. Elles sont souvent confondues ou regroupées sous le terme vague d’« authentification ». Cette documentation vise à les clarifier simplement et à montrer comment elles s’articulent dans un flux d’accès typique.

## Objectifs de cette documentation

Cette documentation vous permettra de :
- Comprendre la différence entre identification, authentification et autorisation
- Savoir dans quel ordre ces étapes interviennent et pourquoi
- Distinguer les mécanismes associés (identifiants, preuves, rôles, permissions)
- Découvrir des exemples concrets : connexion utilisateur, API, RBAC
- Appliquer ces concepts dans la conception et l’implémentation de systèmes sécurisés

## Structure de la documentation

1. **[Introduction](./01-introduction.md)** – Définitions, pourquoi les distinguer, analogie
2. **[Concepts fondamentaux](./02-concepts-fondamentaux.md)** – Identification, authentification, autorisation en détail
3. **[Exemples concrets](./03-exemples-concrets.md)** – Flux de connexion, API, rôles et permissions
4. **[Mise en pratique](./04-mise-en-pratique.md)** – Bonnes pratiques, pièges à éviter, intégration

## Pour qui ?

Cette documentation s’adresse à :
- Développeurs web ou backend mettant en place login, sessions ou API sécurisées
- Étudiants en informatique découvrant la sécurité des systèmes
- Architectes ou chefs de projet souhaitant clarifier le vocabulaire IAA (Identification, Authentification, Autorisation)
- Toute personne souhaitant comprendre « qui fait quoi » dans un contrôle d’accès

## Prérequis

- Notions de base sur les applications web (requêtes HTTP, sessions, cookies)
- Idée générale de ce qu’est un « utilisateur » et un « accès » à une ressource
- (Optionnel) Connaissance de OAuth2, JWT ou RBAC pour les exemples avancés

## Vocabulaire utilisé

- **Identification** : déclarer « qui je suis » (ex. login, email, identifiant)
- **Authentification** : prouver que je suis bien qui je prétends être (ex. mot de passe, 2FA)
- **Autorisation** : vérifier si j’ai le droit d’effectuer une action donnée (ex. rôles, permissions)
- **IAA** ou **AAA** : acronymes pour Identification, Authentification, Autorisation (en français ou en anglais)

---

**Note** : En anglais, on parle souvent d’*Authentication* (qui couvre parfois identification + preuve d’identité) et d’*Authorization* (autorisation). En français, on distingue explicitement identification et authentification pour clarifier l’étape « déclarer son identité » et l’étape « prouver son identité ».

---

Liens :

* [Wikipedia – Authentification][1]
* [OWASP – Authentication Cheat Sheet][2]

[1]: https://fr.wikipedia.org/wiki/Authentification
[2]: https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
