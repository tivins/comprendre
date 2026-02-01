# Stratégies et approches

Ce chapitre présente les **stratégies courantes** pour refactoriser du code legacy de façon progressive : Strangler Fig, branche par abstraction, exécution parallèle et feature flags. Chacune a un cas d’usage typique et des compromis.

## 1. Strangler Fig (Strangler Fig Application)

### Idée

Au lieu de remplacer l’application legacy d’un coup, on fait **pousser** la nouvelle application autour d’elle. Les nouvelles fonctionnalités et les routes (ou cas d’usage) migrés sont gérés par le nouveau code ; le legacy ne traite plus que ce qui n’a pas encore été migré. Au fil du temps, le legacy est de moins en moins sollicité ; un jour on le retire.

En pratique : un routeur (ou un point d’entrée unique) envoie chaque requête soit au nouveau système, soit à l’ancien. On migre route par route (ou module par module).

### Quand l’utiliser

- Application monolithique avec des frontières fonctionnelles identifiables (par URL, par commande, par module métier).
- Équipe prête à maintenir deux codebases en parallèle pendant la transition.
- Besoin de livrer continuellement sans « big bang ».

### Limite

Nécessite un point de routage clair (reverse proxy, facade, bus). Si le legacy n’a pas de frontières nettes (tout appelle tout), le routage peut être complexe. Certaines équipes commencent par une facade qui encapsule tout le legacy, puis migrent l’intérieur de la facade par Strangler.

---

## 2. Branche par abstraction (Branch by Abstraction)

### Idée

On introduit une **abstraction** (interface ou classe de base) qui représente le comportement dont on a besoin. L’ancienne implémentation et la nouvelle implémentent cette abstraction. On bascule les appelants de l’ancienne à la nouvelle via configuration, feature flag ou déploiement. Une fois la nouvelle validée partout, on supprime l’ancienne implémentation et, si besoin, l’abstraction (si une seule implémentation reste).

En pratique : `LegacyPaymentService` et `NewPaymentService` implémentent `PaymentService`. Un conteneur ou une factory retourne l’un ou l’autre selon la config ; on déploie d’abord avec l’ancien, puis on active la nouvelle par flag ou par déploiement progressif.

### Quand l’utiliser

- Remplacement d’un composant bien délimité (service de paiement, envoi d’emails, génération de PDF).
- Interface (entrées/sorties) définissable sans trop modifier les appelants.
- Besoin de pouvoir rollback rapidement (revenir à l’ancienne implémentation).

### Limite

L’abstraction doit refléter le comportement réel du legacy ; sinon la nouvelle implémentation ne sera pas substituable. Les tests de caractérisation sur l’ancienne implémentation aident à définir le contrat.

---

## 3. Exécution parallèle (Parallel Run)

### Idée

On fait tourner **ancienne et nouvelle implémentation** en parallèle pour une même requête (ou un même lot). On compare les résultats ; en cas de divergence, on logue, on alerte ou on garde l’ancien résultat selon la stratégie. Une fois la confiance établie (pas de divergences inexpliquées), on bascule sur la nouvelle implémentation et on arrête l’ancienne.

En pratique : utile pour des calculs critiques (facturation, rapports) ou des intégrations (envoi vers un partenaire). Coût : double exécution, donc à réserver aux chemins où la charge et la latence le permettent.

### Quand l’utiliser

- Comportement critique (argent, données réglementaires) où une régression serait inacceptable.
- Nouvelle implémentation réécrite et pas seulement refactorée — on veut valider l’équivalence avant de couper l’ancienne.

### Limite

Coût en performance et en complexité (orchestration, comparaison, gestion des divergences). Pas adapté à tous les chemins ; souvent limité à un sous-ensemble de scénarios.

---

## 4. Feature flags (feature toggles)

### Idée

Un **feature flag** active ou désactive un comportement (nouveau code, ancien code, A/B) via configuration ou base de données, sans redéploiement. En refactoring legacy : on déploie le nouveau code désactivé ; on l’active pour une partie des utilisateurs ou des requêtes ; on observe ; on généralise ou on rollback.

En pratique : utile pour basculer progressivement (par utilisateur, par région, par pourcentage) et pour découpler le déploiement de l’activation. Attention à ne pas accumuler des flags obsolètes — prévoir une vie courte et un nettoyage.

### Quand l’utiliser

- Bascule progressive (canary, pourcentage de trafic).
- Rollback rapide sans redéploiement.
- Expérimentation (nouveau parcours vs ancien).

### Limite

Prolifération de flags = complexité et risque d’incohérence. Nommer clairement, documenter, et prévoir une date de suppression ou un processus de nettoyage.

---

## 5. Choix de la stratégie

| Stratégie | Cas typique | Coût principal |
|-----------|-------------|-----------------|
| Strangler Fig | Migration d’un monolithe par zones fonctionnelles | Maintien de deux codebases, routage |
| Branche par abstraction | Remplacement d’un composant (service, intégration) | Définition de l’interface, double implémentation |
| Exécution parallèle | Validation d’équivalence sur comportement critique | Performance, orchestration, comparaison |
| Feature flags | Bascule progressive et rollback sans redéploiement | Prolifération des flags, discipline de nettoyage |

On peut les combiner : par exemple Strangler pour migrer un module, branche par abstraction pour remplacer un service à l’intérieur du module, et feature flag pour activer le nouveau chemin progressivement.

---

## Prochaines étapes

- **[Exemples concrets](./04-exemples-concrets.md)** : extraction de service, tests sur code non testé, remplacement d’une dépendance
- **[Mise en pratique](./05-mise-en-pratique.md)** : quand refactoriser, pièges, intégration dans le workflow
