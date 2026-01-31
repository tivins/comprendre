# Introduction aux types de tests

## Pourquoi distinguer les types de tests ?

Les tests automatisés ne sont pas tous au même niveau : certains vérifient une fonction isolée, d’autres un module avec sa base de données, d’autres encore le parcours complet d’un utilisateur. Chaque niveau apporte une forme de confiance et a un coût (temps d’exécution, maintenance, flakiness). Savoir de quoi on parle évite les malentendus (« on a des tests » ne dit pas si la régression est couverte) et aide à équilibrer rapidité, stabilité et couverture.

En résumé : **le type de test définit le périmètre (quoi) et le niveau d’isolation (avec quoi)** — et donc le feedback que vous en tirez.

## Ce qu’apporte chaque niveau (en bref)

- **Unitaire** : une petite unité de code (fonction, classe) testée seule, sans vraie BDD ni réseau. Rapide, nombreux, faciles à localiser une régression. Ne garantissent pas que les briques s’assemblent bien.
- **Intégration** : plusieurs composants ensemble (service + repository + BDD réelle ou conteneur). Plus lents, moins nombreux. Vérifient que les interactions et les contrats (SQL, API interne) tiennent la route.
- **Bout en bout (E2E)** : scénario utilisateur complet via l’interface (navigateur, API publique). Les plus lents et les plus fragiles. Donnent confiance sur le flux métier réel ; à garder en petit nombre et ciblés.
- **Contractuels / composant** : selon les définitions, vérification des interfaces entre services (API, messages) ou d’un composant dans un environnement proche de la prod. Utiles en microservices ou front/back découplés.

Chaque niveau répond à une question différente : « cette fonction calcule-t-elle juste ? » (unitaire), « ce service et cette BDD fonctionnent-ils ensemble ? » (intégration), « l’utilisateur peut-il accomplir cette action ? » (E2E).

## Vocabulaire de base

- **Test automatisé** : script qui exécute du code et vérifie des résultats (assertions), sans intervention manuelle.
- **Assertion** : vérification explicite (égalité, exception, appel de méthode). Ex. : `assertEquals(42, computeAnswer())`.
- **Mock** : objet simulé qui enregistre les appels et peut imposer des réponses ; utilisé pour isoler le code sous test de ses dépendances.
- **Stub** : objet qui renvoie des réponses fixes sans vérifier les appels. Souvent utilisé comme « faux » collaborateur.
- **Test flaky** : test qui passe ou échoue de façon non déterministe (timing, ordre, état partagé). À réduire ou supprimer.
- **Couverture** : mesure (lignes, branches, chemins) du code exécuté par les tests. Indicateur utile mais insuffisant seul — des tests nombreux et rapides qui ne ciblent pas les cas à risque donnent une fausse sécurité.

Ces notions sont reprises et illustrées dans les chapitres suivants.

## Ce que les types de tests ne sont pas

- **Pas une obligation de tout tester partout** : le but est la confiance et la maintenabilité à un coût acceptable. Sur-tester peut ralentir les livraisons et figer le code.
- **Pas une hiérarchie de valeur** : un bon test unitaire et un bon test d’intégration répondent à des questions différentes ; l’important est le bon dosage pour le projet.
- **Pas une substitution au raisonnement** : les tests vérifient un comportement attendu ; ils ne remplacent pas la conception ni la revue de code.

## Quand cette distinction compte

- **Choix de stratégie** : décider combien de tests unitaires vs intégration vs E2E, et où investir en priorité.
- **Diagnostic** : un échec en intégration mais pas en unitaire pointe vers l’assemblage ou l’environnement ; un échec uniquement en E2E vers l’interface ou le flux.
- **Performance des builds** : les tests unitaires restent en général rapides ; les E2E et certains tests d’intégration allongent le pipeline — à garder en nombre maîtrisé.

## Prochaines étapes

La suite de la documentation détaille :

- **[Types et pyramide](./02-types-et-pyramide.md)** : définition opérationnelle de chaque type, pyramide des tests et variantes (cône, diamant)
- **[Exemples concrets](./03-exemples-concrets.md)** : services métier, API, BDD, front
- **[Mise en pratique](./04-mise-en-pratique.md)** : quand écrire quoi, pièges courants, intégration dans le workflow

---

**Rappel** : Distinguer les types de tests permet de choisir le bon niveau pour la bonne question et d’équilibrer vitesse, stabilité et confiance.
