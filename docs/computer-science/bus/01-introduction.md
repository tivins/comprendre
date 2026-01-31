# Introduction au bus

## Qu'est-ce qu'un bus ?

Un **bus** est un canal de communication partagé permettant de transférer des **données**, des **adresses** et des **signaux de contrôle** entre plusieurs composants d’un système (processeur, mémoire, contrôleurs, périphériques). Il joue le rôle de « colonne vertébrale » des échanges : au lieu de relier chaque composant à chaque autre par des liaisons dédiées, on regroupe les connexions sur un même support commun.

### Analogie avec les transports

On peut comparer un bus informatique à un **réseau de bus urbain** :

- **Ligne commune** : un même trajet (le bus physique) est utilisé par plusieurs passagers (les composants).
- **Arrêts** : chaque composant est « branché » sur le bus à un endroit donné (adresse, slot, port).
- **Règles de circulation** : un seul véhicule (un seul maître) utilise la voie à un instant donné ; des règles (arbitrage, protocole) définissent qui parle et quand.
- **Contenu transporté** : les passagers ou les colis correspondent aux données ; la destination correspond à l’adresse.

Sans bus, il faudrait une route dédiée entre chaque paire de composants : le câblage deviendrait ingérable. Le bus **mutualise** les connexions et simplifie l’architecture.

## Rôle du bus dans l'architecture

### 1. Réduire la complexité des connexions

Sans bus, pour connecter \( n \) composants en point à point, il faudrait de l’ordre de \( n(n-1)/2 \) liaisons. Avec un bus, une seule voie partagée (ou un petit nombre de voies) suffit pour relier tous les composants.

```
Sans bus (point à point) :     Avec bus :
    [CPU]───[RAM]                  [CPU]
      \   /   \                    |
       \ /     \                   |
    [GPU]───[I/O]     →         [BUS]
       \   /   \                    |
        \ /     \              [RAM][GPU][I/O]...
    [Périphériques]
```

### 2. Standardiser les échanges

Un bus définit :
- **Le format des signaux** (niveaux électriques, débit, codage)
- **Le protocole** (qui envoie quoi, quand, comment répondre)
- **Les règles d’accès** (arbitrage, priorité)

Cela permet de concevoir des composants interchangeables (cartes PCI, périphériques USB) tant qu’ils respectent le standard du bus.

### 3. Séparer les préoccupations

On distingue souvent :
- **Bus système** (ou bus interne) : liaison processeur ↔ mémoire, cache ; très rapide, court.
- **Bus d’extension / E/S** : liaison vers disques, réseau, cartes ; plus lent, plus long, plus générique.

Chaque type de bus est optimisé pour son usage (débit, latence, nombre de composants).

## Les trois « familles » de signaux sur un bus

Conceptuellement, un bus classique (par exemple un bus système type ancien Front Side Bus) regroupe trois catégories de lignes :

| Famille | Rôle | Exemple |
|--------|------|--------|
| **Données (Data)** | Contenu transféré (instructions, données utilisateur) | 32 ou 64 lignes pour un mot de 32 ou 64 bits |
| **Adresses (Address)** | Où lire/écrire (emplacement en mémoire ou registre) | 32 lignes pour 4 Go d’espace d’adressage |
| **Contrôle (Control)** | Type d’opération, synchronisation, arbitrage | Read/Write, Clock, Reset, Requête bus |

En pratique, sur les bus modernes (série, paquets), données et adresses peuvent être **multiplexées** dans le même flux (ex. en-tête + payload), mais la logique « qui parle, où, quoi » reste la même.

## Maître et esclave

- **Maître (master)** : composant qui **initie** un transfert (demande une lecture/écriture, envoie une commande). Ex. : CPU, contrôleur DMA.
- **Esclave (slave)** : composant qui **répond** aux requêtes (fournit ou reçoit des données). Ex. : RAM, disque, capteur.

Sur un bus donné, à un instant donné, un seul maître pilote le bus (d’où la nécessité d’un **arbitrage** quand plusieurs maîtres potentiels coexistent).

## Pourquoi plusieurs bus ?

Un ordinateur ou un système embarqué utilise en général **plusieurs bus** :

1. **Performance** : le processeur a besoin d’un accès très rapide à la mémoire ; on réserve un bus dédié (ou un canal mémoire) pour cela.
2. **Coût et simplicité** : les périphériques lents n’ont pas besoin d’un bus ultra-rapide ; un bus d’E/S (USB, I2C) est moins cher et plus simple.
3. **Évolutivité** : on peut ajouter des cartes (PCI) ou des périphériques (USB) sans modifier le bus interne du processeur.

On parle alors d’**architecture multi-bus** ou en **couches** (bus processeur, bus mémoire, bus E/S).

## Résumé

- Un **bus** est un canal de transmission **partagé** pour données, adresses et contrôle.
- Il **réduit le câblage**, **standardise** les interfaces et permet de **séparer** bus système et bus E/S.
- Les composants sont soit **maîtres** (initiateurs), soit **esclaves** (répondants), avec des règles d’**arbitrage** pour l’accès au bus.
- Les systèmes réels utilisent **plusieurs bus** adaptés à la vitesse et au type de composants.

Dans la suite, nous détaillons les [concepts fondamentaux](./02-concepts-fondamentaux.md) : lignes, largeur, bande passante, arbitrage et protocoles.
