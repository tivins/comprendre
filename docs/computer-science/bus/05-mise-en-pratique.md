# Mise en pratique : choix et conception

## 1. Critères de choix d'un bus

Lors de la conception d’un système (PC, carte embarquée, réseau de capteurs), le choix du bus dépend de plusieurs critères.

### 1.1 Débit et latence

- **Très haut débit, faible latence** (GPU, SSD, mémoire) → PCIe, canal mémoire dédié (DDR).
- **Débit moyen, périphériques externes** → USB 3.x, Thunderbolt.
- **Débit faible, composants sur carte** → I2C, SPI (selon la vitesse requise).

### 1.2 Distance et environnement

- **Sur la même carte** : I2C, SPI, bus parallèle interne.
- **Câble court (1–3 m)** : USB, Ethernet.
- **Environnement bruité (auto, usine)** : CAN, Ethernet industriel (avec blindage / protocoles adaptés).

### 1.3 Nombre de composants et topologie

- **Peu de composants, liaison dédiée** : SPI (un SS par esclave) ou lien point à point (PCIe).
- **Beaucoup d’esclaves sur peu de fils** : I2C (adressage 7/10 bits), CAN (IDs de message).
- **Extension par câble, hot-plug** : USB (hubs, un port par périphérique ou hub).

### 1.4 Coût et simplicité

- **Minimum de broches** : I2C (2 fils), 1-Wire (1 fil).
- **Simplicité logicielle** : SPI (pas d’adressage dans le protocole, horloge maître).
- **Standard grand public, drivers et OS** : USB.

### 1.5 Alimentation et hot-plug

- **Alimentation sur le bus** : USB (5 V), PoE (Ethernet).
- **Brancher/débrancher à chaud** : USB, PCIe (selon implémentation). I2C/SPI en général non prévus pour le hot-plug.

## 2. Bonnes pratiques de conception

### 2.1 Respecter le standard

- **Niveaux électriques** : tensions, courants, résistances de terminaison (ex. I2C pull-up, CAN 120 Ω).
- **Longueur et débit** : respecter les longueurs max et débits recommandés (ex. USB 3, I2C longue distance).
- **Protocole** : séquences START/STOP (I2C), champs d’en-tête (USB, PCIe), CRC si défini.

### 2.2 Gestion des conflits et erreurs

- **Arbitrage** : sur bus multi-maître (I2C, CAN), prévoir la gestion des collisions (retry, backoff).
- **Détection d’erreurs** : CRC, parité, ou mécanismes du protocole (CAN, USB) ; retransmission ou signalisation à la couche supérieure.

### 2.3 Couche logicielle (pilotes, abstraction)

- **Abstraction** : une couche « bus » (driver, HAL) qui gère le protocole ; le reste du logiciel travaille avec des opérations « lire/écrire registre » ou « envoyer paquet ».
- **Blocage vs asynchrone** : pour les E/S lentes (réseau, disque), privilégier des API asynchrones ou non bloquantes pour ne pas figer le système.

### 2.4 Tests et débogage

- **Analyseur de bus** : logique (I2C, SPI, USB) ou protocolaire (USB, PCIe) pour capturer les trames et vérifier le protocole.
- **Simulation** : modèles des composants (mémoire, périphériques) pour tester le contrôleur de bus sans matériel.
- **Scénarios** : cas limites (bus occupé, esclave absent, erreur CRC) pour valider la robustesse.

## 3. Exemple de décision : capteur sur carte embarquée

**Contexte** : ajouter un capteur (température, accéléromètre) sur une carte avec un microcontrôleur.

- **Peu de broches disponibles, plusieurs capteurs** → **I2C** (2 fils partagés, adresses 7 bits).
- **Besoin de débit élevé (flux continu)** → **SPI** (plus rapide, 4 fils par esclave ou partagés avec SS dédiés).
- **Un seul capteur, simplicité** → **SPI** ou **I2C** selon ce que le capteur propose ; souvent I2C pour limiter les broches.

**À faire** : vérifier les niveaux (3,3 V vs 5 V), pull-up I2C (typ. 4,7 kΩ), longueur des traces, et utiliser le driver fourni par le fabricant ou une HAL du MCU.

## 4. Synthèse

| Besoin | Bus typique |
|--------|-------------|
| Carte d’extension PC (GPU, SSD) | PCIe |
| Périphériques externes (clavier, disque) | USB |
| Capteurs / EEPROM sur carte | I2C |
| Mémoire flash, écran, ADC rapide | SPI |
| Réseau capteurs/actionneurs (auto, industrie) | CAN |
| Réseau local | Ethernet |

Le choix du bus est un compromis entre **débit**, **coût** (broches, câbles), **complexité** du protocole, **environnement** et **écosystème** (pilotes, outils). En restant cohérent avec les standards et en isolant le protocole dans une couche dédiée, on facilite l’évolution et la maintenance du système.

---

Retour à l’[introduction](./01-introduction.md) ou au [README](./README.md) du dossier bus.
