# Types et architectures de bus

## 1. Classification selon la fonction

### 1.1 Bus système (ou bus processeur)

**Rôle** : Relier le processeur à la mémoire principale (RAM) et éventuellement au cache. C’est le chemin critique pour les accès mémoire.

**Caractéristiques** :
- **Très rapide** : faible latence, haut débit pour ne pas ralentir le CPU.
- **Court** : limité au boîtier (CPU, chipset, barrettes RAM).
- **Largeur** : souvent 64 bits (ou plus) en données, adresses sur 40 à 52 bits selon l’architecture.
- **Exemples** : Front Side Bus (FSB, historique), bus mémoire DDR (canal mémoire dédié), QPI/UPI (Intel), Infinity Fabric (AMD).

**Évolution** : Sur les architectures modernes, le « bus » processeur–mémoire est souvent un **contrôleur mémoire intégré** au processeur, avec des canaux dédiés (DDR4/DDR5) plutôt qu’un bus partagé classique.

### 1.2 Bus d'E/S (entrées/sorties)

**Rôle** : Connecter le processeur (ou le chipset) aux périphériques : disques, réseau, cartes d’extension, ports USB, etc.

**Caractéristiques** :
- **Plus lent** que le bus système, mais suffisant pour les E/S.
- **Plus long** : peut sortir du boîtier (câbles USB, Ethernet).
- **Standardisé** : PCI, PCI Express, USB, SATA, etc., pour permettre l’interchangeabilité des périphériques.

### 1.3 Bus d'extension

**Rôle** : Permettre d’ajouter des **cartes** (graphique, réseau, stockage) sans modifier la carte mère.

**Exemples** : ISA (historique), PCI, PCI Express (PCIe). Le slot physique et le protocole définissent le bus d’extension.

## 2. Classification selon le mode de transmission

### 2.1 Bus parallèle

**Principe** : Plusieurs bits sont transmis **simultanément** sur autant de lignes (une ligne par bit pour le bus de données).

**Avantages** :
- Débit élevé par cycle (largeur × fréquence).
- Protocole simple pour des liaisons courtes.

**Inconvénients** :
- **Synchronisation** : à haute fréquence, les délais différents entre lignes (skew) limitent la vitesse.
- **Coût** : beaucoup de pistes ou de broches, câbles épais.
- **Longueur** : peu adapté aux longues distances.

**Exemples** : bus mémoire DDR (données en parallèle), ancien bus ISA/PCI (partie parallèle), bus interne CPU–RAM historique.

### 2.2 Bus série

**Principe** : Les bits sont transmis **un par un** (ou en petit nombre de voies, ex. 2 pour différentiel) sur une ou quelques lignes.

**Avantages** :
- Moins de broches, câbles fins, **longue distance** possible.
- Pas de problème de skew entre bits d’un même lien.
- Permet des débits très élevés en augmentant la fréquence sur peu de lignes.

**Inconvénients** :
- **Multiplexage** : adresses, données et contrôle doivent être encodés dans des **trames** (protocole plus complexe).
- **Latence** : sérialisation/désérialisation et traitement des paquets.

**Exemples** : USB, PCIe, SATA, I2C, SPI, CAN, Ethernet (côté câble).

La tendance actuelle est au **série** pour les bus d’extension et d’E/S (PCIe, USB, SATA) et au **parallèle** surtout pour la liaison mémoire immédiate (DDR) où la distance reste très courte.

## 3. Architecture multi-bus (en couches)

Un système typique **empile** plusieurs bus :

```
                    ┌─────────────┐
                    │    CPU      │
                    └──────┬──────┘
                           │ Bus système / contrôleur mémoire
                    ┌──────┴──────┐
                    │   Mémoire   │  (DDR)
                    └─────────────┘

                    ┌─────────────┐
                    │    CPU      │
                    │  ou North   │
                    │   Bridge    │
                    └──────┬──────┘
                           │ Bus rapide (ex. PCIe)
              ┌────────────┼────────────┐
              │            │            │
         ┌────┴────┐  ┌────┴────┐  ┌────┴────┐
         │  GPU    │  │  South  │  │  ...    │
         └─────────┘  │ Bridge  │  └─────────┘
                      └────┬────┘
                           │ Bus E/S (USB, SATA, PCI legacy...)
              ┌────────────┼────────────┐
              │            │            │
         [USB]        [SATA]       [Réseau]
```

- **Niveau 1** : CPU ↔ mémoire (bus système / canal mémoire).
- **Niveau 2** : CPU/chipset ↔ cartes rapides (PCIe) et pont sud (South Bridge).
- **Niveau 3** : Pont sud ↔ périphériques (USB, SATA, Ethernet, etc.).

Un **pont** (bridge) relie deux bus : il traduit les adresses et les protocoles (ex. North Bridge : CPU ↔ mémoire et PCIe ; South Bridge : PCIe ↔ USB, SATA). Sur les PC récents, North Bridge est souvent intégré au CPU.

## 4. Bus dédiés vs bus partagés

### 4.1 Bus partagé

Plusieurs composants **partagent** les mêmes lignes. À un instant donné, un seul maître peut émettre ; l’arbitrage est nécessaire.

- **Exemples** : ancien bus PCI, I2C, bus système classique.
- **Avantage** : peu de lignes, architecture simple.
- **Inconvénient** : le débit est partagé ; les conflits peuvent augmenter la latence.

### 4.2 Liaisons dédiées (point à point)

Chaque lien relie **deux** composants (ex. CPU ↔ GPU en PCIe dédié). Pas d’arbitrage entre tiers ; débit garanti pour cette liaison.

- **Exemples** : PCIe (chaque lien est point à point), liaison directe CPU–mémoire (canal DDR).
- **Avantage** : performance prévisible, pas de contention entre périphériques différents.
- **Inconvénient** : plus de pistes ou de lignes si beaucoup de composants.

Les architectures récentes mélangent souvent **bus partagés** pour les E/S lentes (I2C, USB hub) et **liens point à point** pour les E/S rapides (PCIe, mémoire).

## 5. Synthèse des types

| Critère | Bus système | Bus d'E/S / extension | Parallèle | Série |
|--------|-------------|------------------------|-----------|-------|
| **Fonction** | CPU–RAM | CPU–périphériques | Données sur N lignes | Données en flux sur 1 ou peu de lignes |
| **Vitesse** | Très rapide | Variable | Fort débit sur court | Fort débit possible, longue distance |
| **Exemples** | FSB, DDR, QPI | PCI, PCIe, USB, SATA | DDR, ancien PCI | PCIe, USB, I2C, SPI, CAN |

La suite propose des [exemples concrets](./04-exemples-concrets.md) : PCI/PCIe, USB, I2C, SPI, CAN, Ethernet.
