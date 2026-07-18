---
title: Concevoir un écosystème documentaire multicanal
slug: documentation-ecosystem
summary: "Retour sur la création d’un pipeline documentaire intelligent transformant une base de connaissances MkDocs en un système de publication multicanal compatible avec les chatbots et l’IA : extraction du HTML rendu, synchronisation en deux passes, réécriture automatique des liens, gestion intelligente des diagrammes Mermaid et émergence progressive d’un véritable graphe documentaire exploitable par des agents IA."
stack:
  [
    MkDocs,
    Material for MkDocs,
    Markdown,
    Python,
    Git,
    GitLab CI/CD,
    API,
    Mermaid,
    YAML,
    HTML,
    Commercial chatbot,
    IA,
  ]
order: 1
featured: true
github_repos:
---

# Du guide PDF au système documentaire multicanal

## Vue d’ensemble

Le point de départ était un guide utilisateur PDF d’environ 100 pages : un document statique, difficile à maintenir, rempli de captures d’écran vieillissantes et peu adapté à un produit complexe en évolution constante.

J’ai choisi de ne pas simplement réécrire ce guide. J’ai conçu et construit un nouvel écosystème documentaire fondé sur MkDocs, avec une source unique en Markdown, une architecture de contenu modulaire, des métadonnées structurées et plusieurs canaux de publication.

Le projet compte aujourd’hui environ 118 fichiers Markdown, auxquels s’ajoutent la configuration MkDocs, les ressources multimédias, les références techniques, les scripts de conversion et d’export, ainsi qu’un journal de travail. Il couvre notamment :

- 23 guides d’actions orientés tâches
- des parcours métier complets
- des concepts et cas d’usage
- des guides de dépannage
- des références CSV et OpenAPI
- des blocs de contenu réutilisables
- des diagrammes, vidéos et contenus interactifs
- une chaîne de publication vers un help center externe

Ce n’est donc plus un simple site de documentation. C’est un système de transformation et de diffusion de la connaissance.

---

## Le problème à résoudre

La documentation initiale concentrait une réelle expertise métier, mais son format empêchait de l’exploiter correctement :

- une seule publication monolithique
- duplication et divergence progressives des informations
- navigation limitée
- mises à jour et validations difficiles à tracer
- captures d’écran rapidement obsolètes
- séparation insuffisante entre concepts, actions et références
- faible réutilisabilité des explications communes
- aucune structure directement exploitable par des outils automatisés
- impossibilité de publier proprement le même contenu sur plusieurs canaux

Le problème n’était pas seulement rédactionnel. Il concernait l’architecture de l’information, la maintenance, la publication et la circulation de la connaissance.

---

## Une décision de conception

J’ai abordé ce chantier comme un problème de conception de système.

L’architecture repose sur une chaîne simple :

**Markdown et métadonnées YAML → rendu MkDocs → transformations contrôlées → site, help center et outils IA**

MkDocs constitue la source canonique. Les autres formats ne sont pas rédigés séparément : ils sont produits à partir du même contenu, selon les contraintes de chaque canal.

Cette décision évite de maintenir plusieurs versions concurrentes de la même information et permet d’appliquer au contenu des principes habituellement réservés au logiciel :

- versioning avec Git
- revue des modifications
- build reproductible
- validation stricte
- composants réutilisables
- déploiement automatisé
- suivi des transformations

---

## Une architecture de contenu explicite

Le dépôt est organisé selon la fonction réelle de chaque contenu :

- **actions** : accomplir une tâche précise
- **concepts** : comprendre les objets et règles du système
- **workflows** : suivre un parcours de bout en bout
- **use cases** : relier le produit à un besoin métier
- **reference** : consulter des formats, champs, endpoints ou contraintes
- **troubleshooting** : diagnostiquer et résoudre un problème
- **\_blocks** : réutiliser des avertissements, prérequis et explications transversales

Un même objectif peut être réalisé par plusieurs modalités — interface, CSV ou API. Les pages sont donc conçues pour présenter un objectif commun tout en séparant clairement les étapes, erreurs et références propres à chaque méthode.

Des blocs partagés permettent également de corriger une explication une seule fois et de la répercuter partout où elle est utilisée.

---

## Les métadonnées comme couche de structure

Chaque page peut porter un front matter YAML décrivant sa fonction et ses relations :

```yaml
type: action
canonical_id: actions.example.csv
modalities: [csv]
contexts: [implementation]
status: published
chatbot:
  export: true
related:
  - concepts.example
  - workflows.example
```

Ces métadonnées ne servent pas uniquement à classer les pages. Elles permettent de :

- sélectionner les contenus à publier
- leur attribuer un identifiant stable indépendant de leur chemin
- relier actions, concepts, workflows et références
- piloter les transformations automatiques
- préparer une récupération de contexte plus précise pour les outils IA
- faire évoluer la documentation sans casser ses relations internes

Le contenu devient ainsi à la fois lisible par une personne et interprétable par une machine.

---

## Un pipeline fondé sur le HTML réellement rendu

Le premier prototype d’export pouvait sembler simple : lire le Markdown et l’envoyer vers une API. Cette approche ne fonctionnait pas correctement.

MkDocs enrichit en effet les sources pendant le build :

- résolution des inclusions
- rendu des admonitions
- génération des onglets
- création des ancres
- coloration du code
- transformation des liens
- intégration des composants Material for MkDocs

Le pipeline extrait donc le **HTML généré par MkDocs**, et non le Markdown brut. Le site publié devient une étape de compilation intermédiaire dont le résultat peut ensuite être adapté aux contraintes du help center.

Cette approche garantit que les contenus réutilisables, les extensions Markdown et les composants visuels sont déjà résolus avant la synchronisation.

---

## Une synchronisation en deux passes

La publication vers le help center repose sur un mécanisme en deux passes.

### Première passe : créer ou mettre à jour

Le pipeline :

1. découvre uniquement les contenus explicitement publiables
2. construit le site avec MkDocs en mode strict
3. extrait et nettoie le HTML rendu
4. crée les articles absents
5. met à jour les articles existants
6. mémorise les correspondances entre identifiants canoniques et identifiants distants

Un fichier d’état évite de recréer les mêmes articles à chaque exécution et permet de suivre leur évolution dans le temps.

### Deuxième passe : reconstruire les relations

Une fois tous les identifiants distants connus, le pipeline reprend les articles pour :

- réécrire les liens internes
- remplacer les chemins MkDocs par les URL finales
- préserver les relations entre articles
- associer les contenus connexes
- éviter les liens cassés lors d’une création ou d’un déplacement

Cette seconde passe résout un problème classique de synchronisation : un article ne peut pas pointer correctement vers une cible distante tant que cette cible n’a pas encore été créée.

Les fichiers intermédiaires, tables de résolution, rendus HTML et images générées restent isolés des sources afin de ne pas polluer le dépôt documentaire.

---

## Le cas particulier des diagrammes Mermaid

Les diagrammes Mermaid sont essentiels pour représenter les workflows, décisions, interactions API et relations entre objets. Mais le help center et son chatbot ne les interprètent pas comme MkDocs.

J’ai donc conçu un traitement spécifique :

1. détection automatique des blocs Mermaid
2. calcul d’une empreinte pour identifier les changements
3. génération du diagramme en SVG
4. téléversement de l’image vers la plateforme cible
5. insertion de l’image dans l’article publié
6. conservation d’une représentation textuelle cachée du diagramme dans le HTML

Cette dernière étape est importante. Une personne voit un schéma proprement rendu ; un chatbot ou un agent IA conserve l’accès aux nœuds, relations, libellés et enchaînements qui composent le diagramme.

Le diagramme n’est donc plus seulement une illustration. Il devient un objet documentaire à double représentation : visuelle pour les humains, structurée pour les machines.

---

## L’émergence d’un graphe documentaire

Le projet ne repose pas encore sur une base de données de graphe dédiée. Pourtant, un véritable graphe documentaire commence déjà à émerger grâce à la combinaison de :

- la taxonomie
- l’identifiant canonique de chaque page
- les métadonnées `related`
- les liens internes
- les relations entre concepts, actions et workflows
- la structure logique des diagrammes Mermaid

Chaque page peut être considérée comme un nœud, et chaque lien ou relation déclarée comme une arête.

Cette structure améliore la navigation humaine, mais elle prépare également des usages plus avancés :

- récupération de contexte pour un chatbot
- exploration de sujets connexes
- réponses guidées par les relations métier
- assistants spécialisés pour les intégrations API
- détection des contenus isolés ou insuffisamment reliés
- génération future de parcours documentaires dynamiques

La compatibilité IA ne vient donc pas d’un simple bouton « chatbot ». Elle commence dans la manière dont la connaissance est découpée, nommée et reliée.

---

## Une publication réellement multicanal

À partir d’une même source, le système peut produire plusieurs expériences :

- **MkDocs / GitLab Pages** pour une documentation riche, structurée et navigable
- **chatbot conversationnel** pour interroger la connaissance en langage naturel
- **outils orientés API** pour retrouver un endpoint, comprendre un modèle ou interpréter une erreur
- **contenus vidéo et H5P** pour les explications qui bénéficient d’un format démonstratif ou interactif

Chaque canal impose ses contraintes de rendu, de navigation et de sécurité. Le rôle du pipeline est de préserver la cohérence du fond tout en adaptant la forme.

---

## CI/CD et qualité documentaire

Le dépôt est intégré à GitLab CI/CD et peut être publié par GitLab Pages.

Le build strict sert de premier niveau de contrôle : une erreur de configuration, une référence invalide ou un contenu incompatible doit être détecté avant publication. Le versioning permet ensuite de comprendre quand et pourquoi une page a changé.

Cette approche rend la documentation :

- versionnée
- testable
- reproductible
- portable
- révisable
- publiable automatiquement

Elle crée aussi les conditions nécessaires pour accueillir d’autres contributeurs sans dépendre d’un processus artisanal connu d’une seule personne.

---

## Mon rôle dans le projet

J’ai initié ce changement d’approche et conçu le système de bout en bout.

Mon travail a notamment consisté à :

- décider de remplacer le guide PDF par une architecture docs-as-code
- créer un nouveau dépôt modulaire
- définir la taxonomie et les conventions de rédaction
- concevoir les métadonnées et identifiants canoniques
- rédiger et restructurer les contenus
- intégrer les références CSV, API et OpenAPI
- concevoir le pipeline d’export du HTML rendu
- mettre au point la synchronisation en deux passes
- résoudre la réécriture des liens internes
- concevoir le traitement multiformat de Mermaid
- préparer le contenu à la recherche conversationnelle et aux agents IA
- organiser la publication et la validation par CI/CD

J’ai utilisé les assistants IA comme accélérateurs d’implémentation, notamment pour produire et faire évoluer les scripts, tout en conservant la responsabilité de l’architecture, des règles métier, des tests et des décisions de conception.

---

## Ce que le projet a déjà changé

Sans prétendre mesurer encore son effet sur le support ou l’adoption produit, le changement structurel est tangible :

- un PDF monolithique a laissé place à environ 118 sources modulaires
- la connaissance peut être corrigée et versionnée de manière ciblée
- les contenus communs peuvent être réutilisés
- les parcours UI, CSV et API peuvent coexister sans être confondus
- les liens restent stables entre plusieurs canaux de publication
- les diagrammes sont maintenables comme du code
- une même source peut servir à la lecture humaine et à la récupération par l’IA
- le système peut évoluer sans imposer une réécriture complète

La documentation n’est plus un livrable produit après coup. Elle devient une infrastructure de connaissance liée au fonctionnement du produit.

---

## Prochaines étapes

Les prochaines évolutions concernent notamment :

- étendre progressivement l’export à d’autres familles de contenus
- formaliser l’onboarding des contributeurs et des administrateurs
- renforcer les contrôles automatiques de qualité
- mieux observer les usages et les recherches sans réponse
- exploiter davantage le graphe documentaire
- fournir aux agents IA un contexte plus précis et plus explicable
- rapprocher encore documentation, support, formation et outils développeur

L’objectif reste le même : produire une connaissance maintenable par les équipes, compréhensible par les utilisateurs et exploitable par les machines.
