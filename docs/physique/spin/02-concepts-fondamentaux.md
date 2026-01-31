# Concepts fondamentaux

## Spin et quantification

En physique classique, un moment cinétique (une « rotation ») peut prendre n’importe quelle valeur. En physique quantique, le **spin** est **quantifié** : il ne peut prendre que des **valeurs discrètes**.

### Valeur du spin \(s\)

On caractérise le spin par un **nombre quantique de spin** \(s\) :

- **\(s\) demi-entier** (1/2, 3/2, …) : particules appelées **fermions** (électron, proton, neutron, quark, …).
- **\(s\) entier** (0, 1, 2, …) : particules appelées **bosons** (photon \(s=1\), gluon \(s=1\), Higgs \(s=0\), …).

Le **moment cinétique intrinsèque** (en norme) vaut \(\sqrt{s(s+1)}\,\hbar\), où \(\hbar\) est la constante de Planck réduite. Pour l’électron (\(s = 1/2\)), on obtient \(\frac{\sqrt{3}}{2}\hbar\). L’essentiel à retenir : la valeur \(s\) est **fixe** pour chaque type de particule (tous les électrons ont \(s = 1/2\)).

### Projection du spin (orientation)

Quand on mesure le spin **le long d’une direction** (par exemple un axe défini par un champ magnétique), la **projection** ne peut prendre que des valeurs **discrètes** :

\[
m_s = -s,\ -s+1,\ \ldots,\ +s
\]

Pour l’électron (\(s = 1/2\)), il n’y a que **deux** possibilités : \(m_s = +1/2\) ou \(m_s = -1/2\). On parle souvent de spin « **haut** » (↑) ou « **bas** » (↓) dans cette direction.

| Particule | \(s\) | Valeurs de \(m_s\) | Type |
|-----------|--------|---------------------|------|
| Électron, proton, neutron | 1/2 | +1/2, −1/2 | Fermion |
| Photon | 1 | +1, 0, −1 (en pratique ±1) | Boson |
| Particule de spin 0 | 0 | 0 | Boson |

L’important : on ne peut pas « voir » une orientation intermédiaire ; la mesure donne toujours une des valeurs autorisées (effet quantique).

## Fermions et bosons

### Fermions (spin demi-entier)

Les **fermions** sont les particules de spin **demi-entier** : \(s = 1/2,\ 3/2,\ \ldots\)

- **Exemples** : électron, proton, neutron, quark, neutrino.
- **Principe d’exclusion de Pauli** : deux fermions **identiques** ne peuvent pas occuper le **même état quantique** (même position, même énergie, même spin, etc.). C’est pour cela que deux électrons dans une même orbitale atomique doivent avoir des spins opposés (un ↑, un ↓).
- **Conséquence** : structure des atomes, tableau périodique, stabilité de la matière ; sans exclusion, les électrons « tomberaient » vers le noyau.

### Bosons (spin entier)

Les **bosons** sont les particules de spin **entier** : \(s = 0,\ 1,\ 2,\ \ldots\)

- **Exemples** : photon (\(s=1\)), gluon (\(s=1\)), boson de Higgs (\(s=0\)).
- **Pas d’exclusion** : plusieurs bosons peuvent occuper le **même état** (ex. nombreux photons dans un même mode du laser).
- **Conséquence** : condensation de Bose-Einstein, superfluides, laser ; des phénomènes collectifs impossibles pour les fermions seuls.

### Résumé fermions / bosons

| Type | Spin \(s\) | Exclusion de Pauli ? | Exemples |
|------|------------|----------------------|----------|
| **Fermion** | 1/2, 3/2, … | Oui : deux fermions identiques ne peuvent pas être dans le même état | Électron, proton, neutron |
| **Boson** | 0, 1, 2, … | Non : plusieurs bosons peuvent être dans le même état | Photon, gluon, Higgs |

## Spin et atome

### Rôle dans la structure électronique

Dans l’atome, chaque **orbitale** (région d’espace associée à une énergie et une forme) peut être occupée par **au plus deux électrons**, à condition qu’ils aient des **spins opposés** (un \(m_s = +1/2\), un \(m_s = -1/2\)). Sans le spin, une seule orbitale ne pourrait accueillir qu’un électron ; la chimie et le tableau périodique seraient différents.

**Exemple** : l’orbitale 1s de l’hélium contient deux électrons (spins ↑ et ↓) ; l’atome est stable. Le spin permet donc de « remplir » les couches électroniques et d’expliquer les propriétés des éléments.

### Spin nucléaire

Les **noyaux** (protons + neutrons) ont aussi un spin **total** (résultant des spins des nucléons). Ce spin nucléaire est utilisé en **RMN** et **IRM** : on applique un champ magnétique et des ondes radio pour faire « basculer » les spins des noyaux (par ex. hydrogène) et détecter un signal. La suite aborde ce lien spin–magnétisme et les applications.

## Résumé

- **Spin** : propriété quantique **quantifiée** ; valeur \(s\) (demi-entier → fermion, entier → boson).
- **Projection** \(m_s\) : dans une direction, valeurs discrètes (ex. +1/2 et −1/2 pour l’électron) ; on parle de spin « haut » ou « bas ».
- **Fermions** (spin 1/2, …) : principe de Pauli ; deux électrons par orbitale avec spins opposés.
- **Bosons** (spin 0, 1, …) : pas d’exclusion ; plusieurs particules dans le même état possible.
- **Atome** : le spin structure les couches électroniques ; les **noyaux** ont aussi un spin (utilisé en RMN/IRM).

La suite aborde les **propriétés et la mesure** du spin (moment magnétique, RMN/IRM en bref) puis des exemples concrets d’application.
