# Bus (système de transmission de données)

## Introduction

Un **bus** est un sous-système de communication qui permet de transférer des données entre les composants d’un ordinateur ou d’un système embarqué. Il constitue l’infrastructure physique et logique sur laquelle reposent les échanges entre le processeur, la mémoire, les contrôleurs et les périphériques.

## Objectifs de cette documentation

Cette documentation vous permettra de :
- Comprendre ce qu’est un bus et son rôle dans l’architecture matérielle
- Maîtriser les concepts fondamentaux : lignes (données, adresses, contrôle), largeur, arbitrage
- Distinguer les types de bus (système, E/S, série, parallèle) et leurs usages
- Découvrir des exemples concrets : bus système (FSB, mémoire), PCI, USB, I2C, SPI
- Savoir choisir et concevoir des interfaces basées sur un bus

## Structure de la documentation

1. **[Introduction au bus](./01-introduction.md)** – Définition, analogie, rôle dans l’architecture
2. **[Concepts fondamentaux](./02-concepts-fondamentaux.md)** – Lignes, largeur, bande passante, arbitrage, protocoles
3. **[Types et architectures](./03-types-et-architectures.md)** – Bus système, bus d’E/S, série vs parallèle
4. **[Exemples concrets](./04-exemples-concrets.md)** – PCI, USB, I2C, SPI, CAN, Ethernet
5. **[Mise en pratique](./05-mise-en-pratique.md)** – Critères de choix, conception, bonnes pratiques

## Pour qui ?

Cette documentation s’adresse à :
- Étudiants en informatique ou en électronique découvrant l’architecture des ordinateurs
- Développeurs embarqués ou système travaillant avec des bus (I2C, SPI, USB)
- Architectes matériel/logiciel ayant besoin de notions sur les interfaces de communication
- Toute personne souhaitant comprendre comment les données circulent dans un système

## Prérequis

- Notions de base en binaire et en représentation des données
- Idée générale du rôle du processeur, de la mémoire et des périphériques
- (Optionnel) Bases en électronique numérique pour les bus bas niveau

## Vocabulaire utilisé

- **Bus** : canal de transmission partagé entre plusieurs composants
- **Maître (master)** : composant qui initie un transfert sur le bus
- **Esclave (slave)** : composant qui répond aux requêtes du maître
- **Arbitrage** : mécanisme qui détermine qui peut utiliser le bus à un instant donné

---

**Note** : Le terme « bus » est utilisé à la fois pour les bus internes (CPU–RAM, chipset) et pour les bus d’extension ou d’E/S (PCI, USB). La documentation couvre les deux aspects avec un focus sur les concepts réutilisables en logiciel et en conception de systèmes.

---

Liens :

* [Wikipedia – Bus (informatique)][1]
* [Computer Hope – Bus][2]

[1]: https://fr.wikipedia.org/wiki/Bus_(informatique)
[2]: https://www.computerhope.com/jargon/b/bus.htm
