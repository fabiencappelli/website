---
title: Traceability Explorer
slug: traceability-explorer
summary: "Un outil Python qui part d’une commande d’achat, reconstruit récursivement une chaîne de traçabilité répartie entre plusieurs environnements API, génère un graphe Mermaid explicable et signale les incohérences de données avant validation."
stack: [Python, REST API, JSON, Mermaid, Graph Traversal, Data Validation, OOP]
order: 0
featured: true
github_repos:
---

# Traceability Explorer : reconstruire et valider une chaîne de traçabilité

## Vue d’ensemble

Traceability Explorer est un outil Python conçu pour automatiser l’analyse de tests de traçabilité réalisés sur une plateforme SaaS.

Le point de départ est une commande d’achat. À partir de cette transaction, l’outil explore plusieurs environnements API, retrouve les événements associés, puis suit récursivement toutes les références vers les lots, articles, sites et autres transactions.

L’objectif n’est pas seulement de collecter des objets. Il s’agit de reconstruire une histoire cohérente :

- qu’est-ce qui a été commandé ?
- qu’est-ce qui a été reçu ?
- quels lots ont été transformés ?
- quels lots ont été produits ou expédiés ?
- les articles correspondent-ils entre la commande et son exécution ?
- la continuité des lots est-elle démontrable ?
- les sites et transactions suivent-ils la structure attendue ?

Le résultat prend la forme d’un graphe Mermaid accompagné de contrôles explicites. L’outil doit permettre de détecter les problèmes avant une validation humaine et de réduire un travail d’investigation pouvant mobiliser environ quatre heures par test.

---

## Le problème initial

La validation d’un test fournisseur était essentiellement manuelle.

Il fallait naviguer entre plusieurs environnements, rechercher une commande, retrouver les événements susceptibles de lui être liés, ouvrir chaque objet, relever ses identifiants, puis recommencer avec les objets nouvellement découverts.

Cette méthode présentait plusieurs limites :

- exploration lente et répétitive
- risque d’oublier un objet indirectement référencé
- difficulté à distinguer une véritable transformation d’une simple continuité de lot
- représentation incomplète des relations plusieurs-à-plusieurs
- détection tardive des incohérences
- dépendance à la connaissance informelle du modèle de données
- résultat difficile à transmettre ou à reproduire

Une chaîne peut sembler correcte lorsqu’on examine ses objets séparément tout en étant incohérente dans son ensemble.

---

## Une chaîne répartie entre deux environnements

L’exploration commence dans l’environnement du donneur d’ordre, appelé ici **T0**. C’est là que se trouve la commande d’achat servant de point d’entrée.

Les événements d’exécution sont ensuite recherchés dans l’environnement du fournisseur, appelé **T1**.

Le programme utilise donc deux connexions API distinctes :

1. **connexion T0** : récupérer la commande, ses articles et les sites concernés
2. **connexion T1** : explorer les réceptions, transformations, expéditions et objets associés

Cette séparation est importante. Un même processus métier traverse plusieurs espaces d’autorisation, et chaque information doit être recherchée dans le contexte où elle est effectivement accessible.

La configuration des connexions est externalisée dans un fichier `.env`, afin de ne jamais placer d’identifiants sensibles dans le code.

---

## Le modèle métier

L’outil traduit le modèle de traçabilité en plusieurs familles d’objets.

### Transactions

Les transactions fournissent le contexte administratif :

- commande d’achat
- ordre de travail
- autres références commerciales ou logistiques

### Événements

Les événements représentent ce qui arrive physiquement aux marchandises :

- **Receiving** : réception d’un lot
- **Transforming** : consommation de lots d’entrée et création de lots de sortie
- **Shipping** : expédition d’un lot

### Données de référence

La chaîne s’appuie également sur :

- les articles
- les lots
- les sites

Un événement peut référencer plusieurs transactions, et une transaction peut apparaître dans plusieurs événements. La relation n’est donc pas un simple arbre : elle est plusieurs-à-plusieurs.

Les transformations ajoutent une autre dimension. Elles ne se contentent pas de relier un événement à un lot ; elles décrivent le passage d’un ou plusieurs lots d’entrée vers un ou plusieurs lots de sortie.

---

## Pourquoi l’exploration doit être récursive

Une exploration limitée aux événements directement liés à la commande ne suffit pas.

Chaque objet découvert peut contenir de nouvelles références :

- une commande référence des articles et des sites
- un événement référence des transactions et des objets traçables
- un lot référence un article
- une transformation relie des lots d’entrée et de sortie
- une transaction peut conduire à d’autres événements

Traceability Explorer maintient donc une file d’objets à examiner. Chaque nouvelle référence est ajoutée à cette file, puis récupérée et analysée à son tour. L’exploration continue jusqu’à ce qu’aucun nouvel objet ne soit découvert.

Le principe est proche d’un parcours en largeur de graphe :

1. récupérer la commande d’entrée
2. extraire ses premières références
3. retrouver les événements candidats dans l’environnement fournisseur
4. filtrer localement lorsque les capacités de recherche distantes ne suffisent pas
5. ajouter tous les objets référencés à la file d’exploration
6. récupérer chaque objet encore inconnu
7. répéter jusqu’à épuisement

Des index en mémoire empêchent les appels inutiles et évitent les boucles lorsqu’un objet est rencontré plusieurs fois.

---

## Résoudre dynamiquement les objets traçables

Une référence d’objet traçable ne permet pas toujours de savoir immédiatement si elle désigne un lot ou un article.

Le moteur de résolution tente donc d’identifier le type réel de l’objet, puis l’enregistre dans le modèle approprié.

Cette distinction est essentielle :

- un **article** décrit ce qu’est un produit
- un **lot** représente une occurrence traçable de ce produit

Utiliser directement des articles dans les événements peut donner l’apparence d’une chaîne complète tout en supprimant la continuité physique nécessaire à une véritable traçabilité.

L’outil signale donc explicitement les situations dans lesquelles des articles sont utilisés comme objets traçables à la place de lots.

---

## Un lot n’est pas un nœud unique

L’une des principales difficultés du projet vient de la représentation des lots.

Un même lot peut apparaître plusieurs fois dans la chaîne :

- comme lot reçu
- comme entrée d’une transformation
- comme sortie d’une transformation
- comme lot expédié
- dans plusieurs événements successifs

Fusionner toutes ces apparitions dans un seul nœud Mermaid ferait disparaître l’ordre des opérations et produirait des raccourcis trompeurs.

Traceability Explorer crée donc une **occurrence distincte pour chaque rôle du lot dans un événement**.

Chaque occurrence possède :

- un identifiant graphique unique
- le lot auquel elle correspond
- l’événement dans lequel elle apparaît
- son rôle
- sa position temporelle

Le graphe peut ainsi représenter à la fois l’identité stable du lot et ses différents usages dans le temps.

---

## Distinguer transformation, continuité et référence

Le graphe utilise plusieurs types de liens.

### Transformation

Une flèche pleine représente une transformation réelle :

```text
lot d’entrée --> lot de sortie
```

La matière ou le produit change d’état dans un événement de transformation.

### Continuité

Un lien de continuité relie deux occurrences du même lot :

```text
lot reçu --- lot expédié
```

Le lot reste identique, mais il apparaît dans deux moments ou deux rôles différents.

Les occurrences considérées comme des sources sont notamment :

- un lot reçu
- un lot produit par une transformation

Les occurrences considérées comme des usages sont notamment :

- un lot expédié
- un lot consommé par une transformation

Pour chaque usage, l’algorithme recherche la source antérieure la plus plausible. Cette règle permet de conserver une continuité chronologique sans confondre passage, stockage et transformation.

### Référence

Un troisième style de lien représente les relations documentaires ou descriptives :

- événement vers transaction
- lot vers article
- transaction vers commande

Ces liens expliquent le contexte sans être interprétés comme un mouvement physique.

---

## Les contrôles produits

Le premier ensemble de validations porte sur des règles simples mais structurantes.

### Structure des sites

L’outil vérifie si les sites associés à la commande et aux événements suivent le motif attendu entre émetteur, origine et destination.

Il ne prend pas automatiquement une décision métier : il présente les écarts pour examen.

### Articles ou lots

Un avertissement est généré lorsqu’un événement utilise un article à la place d’un lot comme objet traçable.

### Correspondance entre commande et exécution

Les articles commandés sont comparés aux articles retrouvés dans les lots reçus, transformés ou expédiés.

Le but est de repérer :

- un article commandé absent de l’exécution
- un article exécuté sans correspondance claire dans la commande
- une confusion entre identifiant d’article et identifiant de lot

### Continuité des lots

Chaque usage d’un lot doit pouvoir être relié à une source antérieure crédible.

Une absence de source, une chronologie incohérente ou un fragment isolé devient immédiatement visible dans le graphe et le rapport de validation.

---

## Une visualisation Mermaid explicable

Le rendu Mermaid est organisé en sous-graphes distincts :

- commandes
- réceptions
- transformations
- expéditions

Cette structure permet de lire le diagramme comme une séquence d’opérations plutôt que comme une simple carte d’objets.

Les nœuds affichent uniquement les informations nécessaires à l’analyse, tandis que les styles de liens distinguent :

- les transformations
- la continuité d’un même lot
- les références administratives

Le choix de Mermaid répond à plusieurs besoins :

- produire un résultat textuel et versionnable
- permettre une inspection rapide
- faciliter le partage d’un cas de test
- conserver une représentation reproductible
- rendre les anomalies visibles sans imposer une interface graphique complexe

Une sortie JSON de diagnostic peut compléter le fichier `.mmd` afin d’inspecter les objets collectés et les décisions prises par l’algorithme.

---

## Architecture du programme

Le projet est organisé autour de plusieurs couches.

### Connexions et rôles

`ApiConnection` et `TenantRole` décrivent les environnements disponibles et empêchent de mélanger les responsabilités de T0 et T1.

### Modèle de domaine

Des classes représentent les objets principaux :

- commandes et ordres de travail
- transactions
- réceptions, transformations et expéditions
- lots, articles et sites

Des énumérations décrivent notamment :

- les types d’événements
- les types d’entités traçables
- les rôles des occurrences de lots
- les types de liens Mermaid

### Contexte d’exploration

`ExplorationContext` centralise :

- les connexions
- les objets déjà récupérés
- la file de travail
- les occurrences de lots
- les liens de continuité
- les avertissements et résultats de validation

### Services

Des services séparés prennent en charge :

- l’exploration récursive
- la résolution des références
- la reconstruction des occurrences
- les contrôles de cohérence
- la génération Mermaid
- l’export de diagnostic

Cette séparation rend le programme plus facile à tester et permet d’adapter les règles métier sans réécrire le moteur d’exploration.

---

## État actuel

Une première V1 exécutable a été assemblée.

Elle est structurée pour :

- charger deux connexions depuis la configuration
- récupérer une commande comme point de départ
- explorer récursivement les objets liés
- construire un contexte unifié en mémoire
- produire un diagramme Mermaid
- générer une sortie JSON de diagnostic
- signaler les premières incohérences de sites, d’articles et de lots

Le projet reste un prototype technique. Les adaptateurs au schéma exact des différentes versions API, les tests automatisés et la présentation finale du rapport doivent encore être consolidés.

Il ne corrige aucune donnée et ne remplace pas le jugement de la personne chargée de la validation. Son rôle est de rendre l’investigation plus rapide, plus complète et plus explicable.

Le gain d’environ quatre heures par test constitue donc un objectif opérationnel à confirmer, et non encore une mesure établie.

---

## Ce que ce projet démontre

Traceability Explorer réunit plusieurs dimensions de mon travail :

- traduire un modèle métier complexe en objets logiciels
- explorer récursivement un graphe distribué entre plusieurs API
- travailler avec des relations plusieurs-à-plusieurs
- gérer l’identité et les occurrences temporelles d’une même entité
- concevoir des contrôles explicables plutôt qu’une boîte noire
- produire une visualisation utile à la décision
- automatiser une tâche opérationnelle née d’un besoin réel

Le projet montre aussi comment je travaille lorsque les données réelles ne correspondent pas à un modèle théorique parfaitement propre : je conserve les ambiguïtés, je les rends visibles et je fournis à la personne qui valide les éléments nécessaires pour décider.

---

## Prochaines étapes

Les évolutions prévues concernent notamment :

- stabiliser les adaptateurs API
- ajouter des tests unitaires et des jeux de données synthétiques
- enrichir les contrôles de chronologie
- détecter plus systématiquement les cycles et fragments déconnectés
- produire un rapport HTML lisible en complément de Mermaid
- distinguer les erreurs bloquantes des avertissements
- comparer plusieurs exécutions d’un même test
- mesurer le temps réellement économisé
- faciliter l’ajout de nouvelles règles sans modifier le moteur principal

L’objectif final est de transformer une investigation artisanale en une analyse reproductible, traçable et assistée — tout en laissant la décision finale à un être humain.
