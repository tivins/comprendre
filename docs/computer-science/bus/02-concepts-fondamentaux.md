# Concepts fondamentaux du bus

## 1. Les lignes du bus

Un bus est constitué de **lignes** (conducteurs, pistes ou canaux logiques) qui transportent des signaux. On distingue trois familles de lignes.

### 1.1 Lignes de données (Data Bus)

**Rôle** : Transporter le contenu effectif du transfert (instructions, données utilisateur, paquets).

- **Bus parallèle** : une ligne par bit ; un mot de 32 bits utilise 32 lignes en parallèle.
- **Bus série** : les bits sont envoyés un par un sur une (ou deux) lignes ; données et adresses sont souvent multiplexées dans des trames (en-tête + payload).

**Largeur en bits** : sur un bus parallèle, le nombre de lignes de données détermine la **largeur du bus** (ex. 32 bits, 64 bits). Plus la largeur est grande, plus on transfère de données par cycle d’horloge.

### 1.2 Lignes d'adresses (Address Bus)

**Rôle** : Indiquer **où** lire ou écrire (emplacement en mémoire, registre, identifiant de périphérique).

- Sur un bus mémoire classique : \( n \) lignes d’adresse permettent d’adresser \( 2^n \) emplacements (ex. 32 lignes → 4 Go).
- Sur un bus paquetisé (USB, PCIe) : l’« adresse » est dans l’en-tête du paquet (adresse logique, ID de fonction, etc.).

Les lignes d’adresse sont en général **unidirectionnelles** (du maître vers les esclaves).

### 1.3 Lignes de contrôle (Control Bus)

**Rôle** : Piloter le type d’opération et la synchronisation. Exemples typiques :

| Signal | Rôle |
|--------|------|
| **Read / Write** | Lecture ou écriture |
| **Clock** | Cadencement des transferts (bus synchrones) |
| **Reset** | Réinitialisation |
| **Interrupt** | Demande d’attention (IRQ) |
| **Bus Request / Grant** | Demande et attribution du bus (arbitrage) |
| **Valid / Acknowledge** | Données valides, accusé de réception |

Sur un bus série, ces informations sont encodées dans le **protocole** (champs de la trame) plutôt que sur des lignes physiques dédiées.

## 2. Largeur du bus et bande passante

### 2.1 Largeur (en bits)

La **largeur** du bus de données est le nombre de bits transférés **en un cycle** (bus parallèle) ou par unité de temps (bus série).

- **Bus 32 bits** : 4 octets par cycle.
- **Bus 64 bits** : 8 octets par cycle.

Pour un bus série, on parle plutôt de **débit** (bits par seconde, ex. Mbit/s, Gbit/s).

### 2.2 Fréquence et bande passante

Pour un bus **synchrone** (cadencé par une horloge) :

\[
\text{Bande passante} \approx \text{largeur (bits)} \times \text{fréquence (Hz)}
\]

Exemple théorique : bus 64 bits à 100 MHz → \( 64 \times 100 \times 10^6 \) bits/s = 800 Mbit/s (sans tenir compte des cycles d’adresse ou de contrôle). En pratique, le débit utile est inférieur (overhead, arbitrage, attentes).

Pour un bus **série** : bande passante ≈ débit nominal (ex. USB 3.0 : 5 Gbit/s), moins l’overhead protocole.

## 3. Bus synchrone vs asynchrone

### 3.1 Bus synchrone

- Tous les transferts sont **cadencés par une horloge** commune.
- À chaque front (montant ou descendant), une phase du protocole s’exécute (adresse, données, etc.).
- **Avantage** : conception simple, prédictible.
- **Inconvénient** : la fréquence est limitée par le composant le plus lent sur le bus.

### 3.2 Bus asynchrone

- Pas d’horloge globale ; la progression est pilotée par des **signaux de handshake** (Requête / Accusé de réception).
- Chaque transfert se termine quand l’autre partie a répondu.
- **Avantage** : des composants de vitesses différentes peuvent coexister.
- **Inconvénient** : protocole et circuits plus complexes.

## 4. Arbitrage du bus

Quand **plusieurs maîtres** peuvent initier des transferts (CPU, DMA, autre contrôleur), un seul peut utiliser le bus à la fois. L’**arbitrage** décide qui obtient le bus.

### 4.1 Principes courants

- **Priorité fixe** : un maître a toujours priorité sur un autre (simple, risque de famine pour les maîtres de faible priorité).
- **Rotation (round-robin)** : chacun obtient le bus à tour de rôle (équitable, moins prévisible).
- **À la demande** : le maître qui demande le bus peut l’obtenir selon des règles (priorité, ancienneté de la requête).

### 4.2 Signaux typiques

- **Bus Request (BR)** : un maître demande le bus.
- **Bus Grant (BG)** : l’arbitre accorde le bus à un maître.

L’arbitre peut être un composant dédié (contrôleur de bus) ou intégré au processeur/chipset.

## 5. Protocole et trames (bus série / paquetisé)

Sur les bus **série** ou **paquetisés** (USB, PCIe, Ethernet), il n’y a plus de lignes séparées données/adresses/contrôle. Tout est **multiplexé** dans des **trames** ou **paquets** :

- **En-tête** : adresse, type de transaction, contrôle d’erreur (CRC), etc.
- **Payload** : données utiles.
- **Parfois** : accusé de réception, statut.

Le **protocole** définit le format des trames, les séquences d’échange (requête → réponse) et la gestion des erreurs (retransmission, signalisation).

## 6. Résumé des concepts

| Concept | Description |
|--------|-------------|
| **Lignes données** | Contenu transféré ; largeur = bits par cycle (parallèle) ou débit (série). |
| **Lignes adresses** | Où lire/écrire ; \( n \) lignes → \( 2^n \) emplacements (ou équivalent logique). |
| **Lignes contrôle** | Type d’opération, horloge, arbitrage, handshake. |
| **Largeur** | Nombre de bits transférés en un cycle (parallèle). |
| **Bande passante** | Débit utile du bus (bits/s). |
| **Synchrone / asynchrone** | Cadencé par une horloge vs handshake. |
| **Arbitrage** | Règles pour attribuer le bus quand plusieurs maîtres coexistent. |
| **Protocole** | Format des trames et règles d’échange (surtout pour bus série/paquetisé). |

La suite présente les [types et architectures](./03-types-et-architectures.md) de bus : bus système, bus d’E/S, série vs parallèle.
