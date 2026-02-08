# Bus applicatif (Message Bus)

## Introduction

Un **bus applicatif** (ou **message bus**) est un mécanisme logiciel qui permet de faire circuler des **messages** (commandes, événements, requêtes) entre les composants d’une application sans les faire dépendre directement les uns des autres. Il joue le rôle de « colonne vertébrale » des échanges côté application : au lieu d’appeler chaque service ou handler manuellement, on envoie un message sur le bus, qui le route vers le bon destinataire.

## Objectifs de cette documentation

Cette documentation vous permettra de :
- Comprendre ce qu’est un bus applicatif et son rôle dans l’architecture logicielle
- Maîtriser les concepts : messages, handlers, middleware, découplage
- Distinguer les types de bus (command bus, event bus) et leur usage en DDD
- Découvrir des exemples concrets en PHP avec le Domain-Driven Design
- Savoir mettre en place un bus dans un projet existant

## Structure de la documentation

1. **[Introduction au bus applicatif](./01-introduction.md)** – Définition, analogie avec le bus matériel, rôle dans l’architecture
2. **[Concepts fondamentaux](./02-concepts-fondamentaux.md)** – Messages, handlers, middleware, enveloppe
3. **[Types et usage](./03-types-et-usage.md)** – Command bus, event bus, intégration DDD
4. **[Exemples concrets](./04-exemples-concrets.md)** – PHP, Symfony Messenger, CQRS, agrégats
5. **[Mise en pratique](./05-mise-en-pratique.md)** – Choix d’implémentation, bonnes pratiques, pièges à éviter


## Vocabulaire utilisé

- **Bus applicatif** : canal logiciel partagé pour envoyer et router des messages (commandes, événements)
- **Message** : objet porteur d’une intention (commande) ou d’un fait (événement)
- **Handler** : composant qui traite un type de message donné
- **Middleware** : couche exécutée avant/après le traitement d’un message (logging, transaction, etc.)

---

**Note** : Le bus applicatif s’inspire du concept de bus matériel (transfert de données entre composants), mais il s’agit d’un mécanisme **côté application** : pas de câbles, uniquement des appels de code et éventuellement des files d’attente pour le traitement asynchrone.

---

Liens :

* [Symfony Messenger](https://symfony.com/doc/current/messenger.html)
* [Tactician (Command Bus PHP)](https://tactician.thephpleague.com/)
* [Martin Fowler – Event-Driven Architecture](https://martinfowler.com/articles/201701-event-driven.html)
