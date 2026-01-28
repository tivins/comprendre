# Mise en Pratique des ADR

Ce guide pratique vous aidera à intégrer les Architecture Decision Records dans votre workflow de développement et à les maintenir efficacement.

## Workflow de création d'un ADR

### Étape 1 : Identifier le besoin

**Quand créer un ADR ?**

Créez un ADR quand vous vous posez une de ces questions :
- "Pourquoi avons-nous fait ce choix ?"
- "Quelles alternatives avons-nous considérées ?"
- "Quels étaient les compromis ?"
- "Pourrait-on faire différemment maintenant ?"

**Signaux d'alerte** :
- Discussion technique qui dure plus de 30 minutes
- Désaccord dans l'équipe sur une approche
- Choix qui affecte plusieurs équipes
- Décision qui sera difficile à changer plus tard

### Étape 2 : Rédiger l'ADR pendant la décision

**⚠️ Erreur commune** : Attendre après la décision pour documenter.

**✅ Bonne pratique** : Rédiger l'ADR **pendant** la discussion de la décision.

**Pourquoi ?**
- Le contexte est encore frais dans les esprits
- Les alternatives sont encore discutées
- Les compromis sont identifiés
- Moins de temps perdu à se souvenir après coup

**Workflow recommandé** :
1. Ouvrir un template ADR au début de la discussion
2. Remplir le contexte au fur et à mesure
3. Noter les alternatives discutées en temps réel
4. Finaliser la décision et les conséquences après la décision

### Étape 3 : Révision et validation

**Qui doit réviser ?**
- L'auteur de l'ADR
- Les décideurs
- Les personnes impactées
- Au moins un pair (code review)

**Checklist de révision** :
- [ ] Le contexte est clair et complet
- [ ] La décision est sans ambiguïté
- [ ] Les alternatives principales sont documentées
- [ ] Les conséquences sont réalistes (positives ET négatives)
- [ ] Le plan de mise en œuvre est actionnable
- [ ] Les métriques à surveiller sont définies

### Étape 4 : Publication et communication

**Où publier ?**
- Dans le dépôt Git du projet (dans `docs/adr/`)
- Accessible à toute l'équipe
- Versionné avec le code

**Comment communiquer ?**
- Partager le lien dans le canal Slack/Teams de l'équipe
- Mentionner dans la réunion d'équipe
- Ajouter au README du projet
- Référencer dans les PRs liées

## Organisation pratique

### Structure de dossiers

```
mon-projet/
├── docs/
│   ├── adr/
│   │   ├── 0001-utiliser-react-frontend.md
│   │   ├── 0002-adopter-postgresql.md
│   │   ├── 0003-implementer-oauth.md
│   │   ├── README.md (index)
│   │   └── template.md
│   └── README.md (référence vers les ADR)
└── ...
```

### Nommage des fichiers

**Format recommandé** : `NNNN-titre-en-minuscules.md`

**Règles** :
- Numéro séquentiel sur 4 chiffres (0001, 0002, etc.)
- Titre en minuscules avec tirets
- Descriptif mais concis (max 50 caractères)

**Exemples** :
- ✅ `0001-utiliser-react-frontend.md`
- ✅ `0015-remplacer-react-par-vue.md`
- ❌ `ADR-1-react.md` (incohérent)
- ❌ `001-choix-du-framework-frontend-react.md` (trop long)

### Fichier index (README.md)

Créer un fichier `docs/adr/README.md` qui liste tous les ADR :

```markdown
# Architecture Decision Records

Ce dossier contient tous les Architecture Decision Records du projet.

## Liste des ADR

| Numéro | Titre | Statut | Date | Tags |
|--------|-------|--------|------|------|
| [ADR-001](./0001-utiliser-react-frontend.md) | Utiliser React pour le frontend | Accepté | 2024-01-15 | frontend, react |
| [ADR-002](./0002-adopter-postgresql.md) | Adopter PostgreSQL | Accepté | 2024-02-10 | database, postgresql |
| [ADR-003](./0003-implementer-oauth.md) | Implémenter OAuth 2.0 | Accepté | 2024-03-05 | security, oauth |
| [ADR-004](./0004-utiliser-docker.md) | Utiliser Docker | Proposé | 2024-04-01 | infrastructure, docker |

## Comment créer un ADR

1. Copier le template : `cp template.md NNNN-titre.md`
2. Remplir les sections
3. Créer une PR avec l'ADR
4. Obtenir l'approbation de l'équipe
5. Mettre à jour cet index

## Statuts

- **Proposé** : En cours de discussion
- **Accepté** : Décision approuvée et implémentée
- **Rejeté** : Décision rejetée
- **Remplacé** : Remplacé par un autre ADR
```

## Template réutilisable

Créer un fichier `docs/adr/template.md` :

```markdown
# ADR-XXX : [Titre de la décision]

**Date** : YYYY-MM-DD  
**Auteur(s)** : [Nom(s)]  
**Décideur(s)** : [Nom(s)]  
**Consulté(s)** : [Nom(s)]  
**Tags** : [tag1, tag2, tag3]

## Statut
[Proposé | Accepté | Rejeté | Remplacé]

## Contexte

[Pourquoi cette décision est nécessaire ? Quel problème résout-on ?]

## Décision

[Quelle décision a été prise ? Être spécifique et actionnable.]

## Alternatives considérées

### 1. [Alternative 1]
**Avantages** :
- 

**Inconvénients** :
- 

**Pourquoi rejeté** : 

### 2. [Alternative 2]
**Avantages** :
- 

**Inconvénients** :
- 

**Pourquoi rejeté** : 

## Conséquences

### Positives
- 

### Négatives
- 

### À surveiller
- 

## Notes de mise en œuvre

[Plan concret pour implémenter cette décision]

## Références

[Liens vers des documents, articles, ou discussions pertinents]
```

## Intégration dans le workflow

### Intégration avec Git

**Workflow recommandé** :

1. **Créer une branche** : `git checkout -b adr/001-utiliser-react-frontend`
2. **Créer l'ADR** : Créer le fichier dans `docs/adr/`
3. **Mettre à jour l'index** : Ajouter l'entrée dans `README.md`
4. **Commit** : `git commit -m "docs: add ADR-001 for React frontend decision"`
5. **Pull Request** : Créer une PR pour review
6. **Merge** : Une fois approuvé, merger dans `main`

**Avantages** :
- Review par l'équipe
- Historique Git complet
- Discussion dans la PR
- Traçabilité

### Intégration avec les Pull Requests

**Bonnes pratiques** :

1. **Référencer l'ADR dans les PRs** :
   ```
   Cette PR implémente la décision documentée dans ADR-001.
   ```

2. **Créer l'ADR avant la PR** :
   - D'abord créer et approuver l'ADR
   - Ensuite créer la PR d'implémentation

3. **Lier les PRs dans l'ADR** :
   ```markdown
   ## Implémentation
   - PR #123 : Setup React et TypeScript
   - PR #124 : Migration des premières pages
   ```

### Intégration avec les réunions

**Dans les réunions techniques** :

1. **Avant la réunion** : Créer un brouillon d'ADR si une décision importante est prévue
2. **Pendant la réunion** : Remplir l'ADR en temps réel
3. **Après la réunion** : Finaliser et partager l'ADR

**Dans les rétrospectives** :

- Revoir les ADR créés récemment
- Vérifier si les conséquences prévues se sont réalisées
- Identifier les ADR à réviser

## Maintenance des ADR

### Quand mettre à jour un ADR ?

**Mettre à jour quand** :
- Le statut change (Proposé → Accepté)
- La décision est remplacée par une nouvelle
- Des informations importantes manquent
- Des erreurs sont découvertes

**Ne pas mettre à jour** :
- Pour changer l'historique (garder l'historique intact)
- Pour documenter l'implémentation (créer une nouvelle section ou un autre document)

### Gestion des ADR remplacés

**Quand une décision est remplacée** :

1. **Créer un nouvel ADR** qui remplace l'ancien
2. **Mettre à jour l'ancien ADR** :
   ```markdown
   ## Statut
   Remplacé par [ADR-015](./0015-nouvelle-decision.md)
   ```

3. **Référencer l'ancien dans le nouveau** :
   ```markdown
   ## Contexte
   Cet ADR remplace ADR-001 qui documentait l'utilisation de React.
   Le contexte a changé car...
   ```

### Nettoyage et archivage

**ADR obsolètes** :

- Ne pas supprimer les ADR (ils font partie de l'historique)
- Marquer comme "Dépublié" si vraiment plus pertinent
- Garder dans le dépôt Git pour l'historique

**ADR rejetés** :

- Garder les ADR rejetés (ils documentent pourquoi une option n'a pas été choisie)
- Utiles pour éviter de rediscuter les mêmes options

## Outils et automatisation

### Outils en ligne de commande

**adr-tools** (Python) :
```bash
# Installation
pip install adr-tools

# Créer un nouvel ADR
adr new "Utiliser React pour le frontend"

# Lister les ADR
adr list

# Mettre à jour le statut
adr status 001 accepted
```

**adr** (Node.js) :
```bash
# Installation
npm install -g adr

# Créer un ADR
adr create "Utiliser React pour le frontend"

# Lister
adr list
```

### Scripts personnalisés

**Script de création d'ADR** (`scripts/new-adr.sh`) :

```bash
#!/bin/bash

# Usage: ./scripts/new-adr.sh "Titre de la décision"

TITLE=$1
NUMBER=$(ls docs/adr/*.md | wc -l | xargs printf "%04d")
FILENAME="docs/adr/${NUMBER}-$(echo "$TITLE" | tr '[:upper:]' '[:lower:]' | tr ' ' '-').md"

cp docs/adr/template.md "$FILENAME"

# Remplacer les placeholders
sed -i "s/ADR-XXX/ADR-${NUMBER}/g" "$FILENAME"
sed -i "s/\[Titre de la décision\]/$TITLE/g" "$FILENAME"
sed -i "s/YYYY-MM-DD/$(date +%Y-%m-%d)/g" "$FILENAME"

echo "ADR créé : $FILENAME"
echo "Ouvrir avec : code $FILENAME"
```

### Intégration CI/CD

**Validation automatique** :

Créer un script de validation (`scripts/validate-adr.sh`) :

```bash
#!/bin/bash

# Vérifier que tous les ADR ont les sections requises
for file in docs/adr/*.md; do
    if [[ "$file" != "docs/adr/README.md" && "$file" != "docs/adr/template.md" ]]; then
        if ! grep -q "## Statut" "$file"; then
            echo "❌ $file : Section 'Statut' manquante"
            exit 1
        fi
        if ! grep -q "## Contexte" "$file"; then
            echo "❌ $file : Section 'Contexte' manquante"
            exit 1
        fi
        if ! grep -q "## Décision" "$file"; then
            echo "❌ $file : Section 'Décision' manquante"
            exit 1
        fi
    fi
done

echo "✅ Tous les ADR sont valides"
```

**Intégrer dans GitHub Actions** :

```yaml
name: Validate ADR

on:
  pull_request:
    paths:
      - 'docs/adr/**'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Validate ADR
        run: ./scripts/validate-adr.sh
```

## Bonnes pratiques

### 1. Rester concis

**❌ Trop long** :
- ADR de 10 pages avec tous les détails d'implémentation
- Répétitions inutiles
- Informations non pertinentes

**✅ Bon** :
- ADR de 1-2 pages maximum
- Focus sur le "pourquoi", pas le "comment"
- Informations essentielles uniquement

### 2. Être honnête

**❌ Cacher les problèmes** :
- Ne documenter que les avantages
- Ignorer les risques
- Présenter comme une décision parfaite

**✅ Être transparent** :
- Documenter les avantages ET inconvénients
- Reconnaître les risques et incertitudes
- Être honnête sur les compromis

### 3. Maintenir la cohérence

**❌ Formats différents** :
- Chaque ADR dans un format différent
- Numérotation incohérente
- Tags différents pour des concepts similaires

**✅ Standardiser** :
- Utiliser le même template pour tous
- Suivre la numérotation séquentielle
- Utiliser des tags cohérents

### 4. Rédiger pour l'avenir

**❌ Jargon du moment** :
- "On a choisi React parce que c'est hype"
- Références à des personnes qui ne seront plus là
- Contexte implicite

**✅ Clarté durable** :
- Expliquer le contexte de manière compréhensible dans 2 ans
- Utiliser des termes standards
- Documenter les contraintes et exigences

### 5. Intégrer dans le workflow

**❌ ADR comme corvée** :
- Créer les ADR après coup
- Voir ça comme de la bureaucratie
- Ne pas les utiliser

**✅ ADR comme outil** :
- Créer pendant la décision
- Utiliser pour guider les décisions futures
- Référencer dans les discussions

## Pièges à éviter

### Piège 1 : Trop d'ADR

**Symptôme** : ADR pour chaque petite décision

**Solution** : Se concentrer sur les décisions architecturales importantes

### Piège 2 : ADR trop techniques

**Symptôme** : ADR qui ressemble à de la documentation technique

**Solution** : Focus sur le "pourquoi", pas le "comment"

### Piège 3 : ADR non maintenus

**Symptôme** : ADR obsolètes, statuts non mis à jour

**Solution** : Intégrer la maintenance dans le workflow (rétrospectives)

### Piège 4 : ADR ignorés

**Symptôme** : Personne ne lit les ADR, décisions répétées

**Solution** : Référencer les ADR dans les discussions, intégrer dans l'onboarding

### Piège 5 : ADR comme justification

**Symptôme** : Utiliser les ADR pour justifier des mauvaises décisions

**Solution** : Les ADR documentent, ils ne justifient pas. Être prêt à réviser.

## Métriques et suivi

### Métriques à suivre

**Quantitatives** :
- Nombre d'ADR créés par mois
- Temps moyen pour créer un ADR
- Taux d'ADR remplacés (indicateur de changement de contexte)

**Qualitatives** :
- Les nouveaux développeurs comprennent-ils l'architecture grâce aux ADR ?
- Les ADR sont-ils référencés dans les discussions ?
- Les décisions sont-elles mieux documentées qu'avant ?

### Amélioration continue

**Questions à se poser régulièrement** :
- Les ADR sont-ils utiles ?
- Le format est-il adapté ?
- Le workflow est-il trop lourd ?
- Y a-t-il des ADR manquants ?

**Ajustements** :
- Adapter le format si nécessaire
- Simplifier le workflow si trop lourd
- Former l'équipe si les ADR ne sont pas utilisés

## Conclusion

Les Architecture Decision Records sont un outil puissant pour documenter les décisions importantes, mais ils nécessitent :

1. **Un workflow clair** : Savoir quand et comment créer un ADR
2. **Une organisation** : Structure de fichiers et nommage cohérents
3. **Une intégration** : Faire partie du workflow naturel de l'équipe
4. **Une maintenance** : Garder les ADR à jour et pertinents
5. **Une utilisation** : Référencer et utiliser les ADR dans les discussions

Avec ces pratiques, les ADR deviennent un atout pour votre équipe plutôt qu'une charge administrative.

**Prochaines étapes** :
1. Créer la structure de dossiers dans votre projet
2. Créer un template adapté à votre contexte
3. Identifier une première décision à documenter
4. Créer votre premier ADR !
