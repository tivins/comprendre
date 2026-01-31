# Exemples concrets de bus

## 1. PCI et PCI Express (PCIe)

### 1.1 PCI (Parallel PCI)

- **Type** : bus parallèle, partagé, synchrone.
- **Rôle** : cartes d’extension (réseau, son, stockage) sur PC.
- **Caractéristiques** : 32 ou 64 bits de données, 32 bits d’adresse, 33 ou 66 MHz. Arbitrage par priorité (Bus Request / Grant).
- **Usage** : encore présent en mode « legacy » sur certaines cartes mères ; largement remplacé par PCIe.

### 1.2 PCI Express (PCIe)

- **Type** : bus **série**, **point à point** : chaque périphérique a son propre lien (lane) vers le contrôleur.
- **Rôle** : cartes graphiques, SSD NVMe, cartes réseau rapides.
- **Lanes** : 1, 2, 4, 8, 16 lanes par lien. Débit par lane (génération 4) : ~2 Go/s par sens.
- **Protocole** : paquets (Transaction Layer Packet, TLP) avec en-tête (adresse, type, ID) et payload. Couches : physique, liaison, transaction.

```text
Exemple : slot PCIe x16 = 16 lanes en parallèle (16 liens série) pour une carte graphique.
```

## 2. USB (Universal Serial Bus)

- **Type** : bus **série**, **partagé** (topologie en étoile via hubs).
- **Rôle** : périphériques externes (clavier, souris, disques, caméras).
- **Maître** : l’hôte (PC, téléphone) ; les périphériques sont des **esclaves** (sauf USB On-The-Go).
- **Versions** : USB 1.x (1,5 / 12 Mbit/s), USB 2.0 (480 Mbit/s), USB 3.x (5–20 Gbit/s selon génération).
- **Protocole** : trames avec champs (PID, adresse, endpoint, données, CRC). Transferts par « pipes » (flux logiques) entre hôte et périphérique.
- **Avantage** : hot-plug, alimentation sur le bus (5 V), un seul type de connecteur pour de nombreux usages.

## 3. I2C (Inter-Integrated Circuit)

- **Type** : bus **série**, **synchronisé par horloge**, **partagé** (multi-maître possible).
- **Rôle** : communication courte distance sur carte (capteurs, EEPROM, écrans, RTC). Très utilisé en embarqué.
- **Lignes** : **SDA** (données) et **SCL** (horloge), plus masse. Deux fils seulement.
- **Adressage** : 7 ou 10 bits par esclave ; chaque composant a une adresse fixe ou configurable.
- **Débit** : 100 kbit/s (standard), 400 kbit/s (Fast), 1 Mbit/s et plus (Fast+, High speed).
- **Arbitrage** : si deux maîtres émettent en même temps, le bus « AND » logique fait que le 0 l’emporte ; le maître qui voulait envoyer 1 détecte un conflit et abandonne.

```text
Séquence typique : START → adresse esclave (7 bits) + R/W → ACK → données → ACK → ... → STOP
```

## 4. SPI (Serial Peripheral Interface)

- **Type** : bus **série**, **synchronisé**, **full-duplex**.
- **Rôle** : liaison rapide sur carte (mémoires flash, écrans, ADC, capteurs). Embarqué et industrie.
- **Lignes** : **MOSI** (Master Out Slave In), **MISO** (Master In Slave Out), **SCK** (horloge), **SS/CS** (sélection d’esclave par maître). Un signal SS par esclave en général.
- **Maître unique** : pas d’arbitrage ; le maître génère l’horloge et choisit l’esclave via SS.
- **Débit** : très variable, souvent de 1 à 50 Mbit/s et au-delà selon le composant.
- **Avantage** : simple, rapide, full-duplex. Inconvénient : plus de fils que I2C (au moins 4).

## 5. CAN (Controller Area Network)

- **Type** : bus **série**, **différentiel**, **multi-maître**, conçu pour environnements bruités.
- **Rôle** : automobile, industrie, robots (réseau de capteurs et actionneurs).
- **Caractéristiques** : deux fils (CAN_H, CAN_L), débit typique 125 kbit/s à 1 Mbit/s. Accès par **arbitrage** sur l’identifiant du message : plus la priorité est haute (ID numérique bas), plus le message peut « gagner » le bus.
- **Protocole** : trames avec ID, DLC (longueur), données, CRC. Détection d’erreurs et retransmission.

## 6. Ethernet (couche physique / liaison)

- **Type** : bus **série** (sur paire torsadée ou fibre), à l’origine **partagé** (segment coaxial), aujourd’hui le plus souvent **commuté** (switch = liaisons point à point logiques).
- **Rôle** : réseau local (LAN), backbone, liaison machine à machine.
- **Débits** : 10/100/1000 Mbit/s (Cuivre), 1/10/40/100 Gbit/s (Cuivre ou fibre).
- **Protocole** : trames avec adresses MAC, type, payload, CRC. Au-dessus : IP, TCP/UDP, etc. Ethernet définit surtout la couche physique et la trame de liaison.

## 7. Tableau récapitulatif

| Bus   | Type      | Maître(s) | Fils / Lanes     | Usage typique           |
|-------|-----------|-----------|-------------------|--------------------------|
| PCIe | Série, pt-à-pt | 1 par lien | 1–16 lanes        | GPU, SSD, cartes rapides |
| USB   | Série, partagé | Hôte      | 2–4 (data + power)| Périphériques externes   |
| I2C   | Série, partagé | Multi     | 2 (SDA, SCL)      | Capteurs, EEPROM, écrans |
| SPI   | Série, partagé | 1         | 4+ (MOSI, MISO, SCK, SS) | Flash, écrans, ADC   |
| CAN   | Série, partagé | Multi     | 2 (CAN_H, CAN_L)  | Auto, industrie          |
| Ethernet | Série   | N/A (commuté) | 2–4 paires / fibre | Réseau local, WAN    |

Ces exemples illustrent la diversité des bus : **parallèle vs série**, **partagé vs point à point**, **multi-maître vs maître unique**, et l’importance du **protocole** (trames, adressage, arbitrage) pour le bon fonctionnement du système.

La suite propose une [mise en pratique](./05-mise-en-pratique.md) : critères de choix et bonnes pratiques pour concevoir ou utiliser un bus.
