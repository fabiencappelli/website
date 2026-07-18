---
title: "Chatbot Integration v1"
date: 2026-05-15
lang: fr
project: documentation-ecosystem
image:
categories: [documentation]
summary: "Retour sur la création d’un pipeline documentaire intelligent transformant une base de connaissances MkDocs en un système de publication multicanal compatible avec les chatbots et l’IA : extraction du HTML rendu, synchronisation en deux passes, réécriture automatique des liens, gestion intelligente des diagrammes Mermaid et émergence progressive d’un véritable graphe documentaire exploitable par des agents IA."
---

# Repenser un écosystème documentaire : de la documentation statique à un pipeline de publication intelligente

Souvent, la documentation technique est pensée comme une destination finale.
On écrit des pages. On les publie. Elles vivent quelque part dans un portail documentaire plus ou moins élégant. Fin de l’histoire.

Ces dernières semaines, j’ai travaillé sur quelque chose de très différent : créer, et transformer une documentation MkDocs existante en véritable système de publication multi-canal, capable d’alimenter à la fois :

- un site documentaire riche,
- une base de connaissance embarquée dans un chatbot support qui fournit à la fois l'accès aux articles de la documentation et un agent conversationnel intelligent qui se base sur ces articles pour donner ses réponses,
- et, à terme, des outils IA et RAG capables de raisonner sur cette documentation.

Et honnêtement, le moment où la première V1 a fonctionné “pour de vrai” a été un énorme moment de satisfaction technique.

---

## Le vrai problème : MkDocs n’était pas la destination finale

Au départ, le besoin semblait presque banal :

> Publier automatiquement une partie d’une documentation MkDocs dans une plateforme de support client intégrée à un chatbot.

Mais très vite, plusieurs contraintes sont apparues.

Le rendu de MkDocs était déjà très riche :

- snippets Markdown,
- macros,
- admonitions,
- Mermaid,
- tabs,
- ancres,
- blocs API,
- liens internes complexes,
- transformations Material for MkDocs.

Et surtout : tout cela existait déjà dans le HTML généré par MkDocs.

Ce constat a complètement changé l’architecture du projet.

---

## La décision qui a tout débloqué

Le réflexe classique aurait été :

```text
Markdown → HTML → plateforme cible
```

Mais cela aurait obligé à reconstruire toute la logique de rendu.

Au lieu de cela, nous avons pris une autre direction :

```text
mkdocs build
    ↓
HTML déjà rendu par MkDocs
    ↓
Extraction + nettoyage
    ↓
Publication
```

C’est devenu le cœur de tout le pipeline.

MkDocs restait le moteur de rendu canonique.

Le système de publication ne faisait plus de “conversion Markdown”, mais du _post-processing intelligent_ du rendu final.

Et ça change tout.

Parce qu’à partir de là :

- les snippets sont déjà résolus,
- les macros sont déjà exécutées,
- les admonitions existent déjà,
- les ancres existent déjà,
- les liens sont déjà calculés.

La documentation n’est plus du texte brut : c’est déjà une structure riche.

---

## Le moment où tout a commencé à devenir intéressant

Très vite, le projet a cessé d’être _juste_ un export de documentation.

On a commencé à toucher à des problématiques beaucoup plus profondes :

- publication multi-canal,
- normalisation HTML,
- transformation de contenu,
- synchronisation d’articles,
- gestion d’assets,
- réécriture de liens,
- mapping d’identifiants,
- compatibilité chatbot,
- futur graphe documentaire exploitable par IA.

Autrement dit :

la documentation devenait progressivement une couche d’abstraction.

---

## Le problème fascinant des liens internes

L’un des moments les plus intéressants techniquement a été la découverte du comportement des URLs générées par la plateforme cible.

Les URLs finales des articles n’étaient pas connues à l’avance.

Elles étaient générées dynamiquement au moment de la création des articles.

Ce qui signifie qu’un lien interne comme :

```text
../../../concepts/item/
```

ne pouvait pas être correctement converti immédiatement.

Et là, l’architecture a basculé vers quelque chose de beaucoup plus propre : une synchronisation en deux passes.

---

## Architecture finale de la V1

La logique est devenue :

### Première passe

- créer les articles,
- récupérer leurs métadonnées runtime,
- construire une table de correspondance.

### Deuxième passe

- réécrire tous les liens internes,
- injecter les relations entre articles,
- finaliser le contenu.

---

## Le Beacon/chatbot : la réalité des contraintes

Un autre aspect passionnant a été le travail sur le rendu dans le chatbot embarqué.

Et là, la réalité du web moderne a frappé immédiatement :

ce qui fonctionne dans un site complet ne fonctionne pas forcément dans un composant embarqué.

Certaines choses passaient parfaitement :

- tableaux simples,
- blocs de code,
- callouts,
- HTML minimaliste.

D’autres beaucoup moins :

- CSS complexes,
- tabs,
- boutons,
- composants JS,
- certaines classes visuelles.

On a donc commencé à définir une véritable “grammaire HTML sûre” pour les environnements chatbot.

Ce qui est fascinant ici, c’est qu’on revient presque à une forme de sobriété web :

- HTML robuste,
- structure claire,
- sémantique forte,
- dépendance minimale au JavaScript.

Et paradoxalement, cela rend souvent le contenu plus lisible.

---

## Mermaid, ou comment garder des diagrammes “lisibles” par une IA

Un des problèmes les plus intéressants concernait les diagrammes Mermaid.

Le site MkDocs les rendait parfaitement.
Le chatbot embarqué, lui, ne savait pas les interpréter correctement.

Le réflexe classique aurait été simple :

```text
Mermaid
    ↓
SVG
    ↓
Image finale
```

Mais cela posait un problème beaucoup plus subtil.

Une image est parfaite pour les humains.
Beaucoup moins pour un agent IA.

Or, l’objectif n’était pas seulement d’obtenir un joli rendu visuel.
Il fallait aussi préserver la structure logique du diagramme pour nourrir les capacités de recherche et de compréhension du système conversationnel intégré.

La solution finale a été beaucoup plus élégante.

Nous avons mis en place une pipeline qui :

- détecte automatiquement les blocs Mermaid,
- génère les SVG automatiquement,
- upload automatiquement les assets,
- réinjecte les images compatibles dans le HTML final,
- tout en conservant le flow Mermaid original “caché” dans le HTML.

Autrement dit :

- les humains voient un diagramme parfaitement rendu,
- le chatbot voit toujours la structure Mermaid source,
- l’agent IA peut donc continuer à exploiter :
  - les nœuds,
  - les relations,
  - les labels,
  - la logique du flow.

Et ce n’est pas tout.

Comme les diagrammes peuvent devenir nombreux, nous avons ajouté un système de hash permettant de détecter automatiquement si un diagramme a réellement changé.

Ce qui signifie :

```text
Source Mermaid
    ↓
Hash
    ↓
Diagram déjà connu ?
       ├── oui → réutiliser l'image existante
       └── non  → regénérer une image et l'uploader
```

Résultat :

- pas d’upload inutile,
- pas de duplication d’assets,
- synchronisation beaucoup plus rapide,
- versionnement implicite des diagrammes.

C’est probablement l’un des passages du projet où la documentation a cessé d’être simplement “du contenu affiché”.

Le diagramme devenait simultanément :

- un objet visuel,
- un objet documentaire,
- un objet logique,
- et un objet exploitable par IA.

Et ça, franchement, c’était un très beau moment d’architecture.

---

## Ce que le projet est en train de devenir

C’est probablement la partie la plus intéressante.

Au départ :

> “Exporter de la doc.”

Aujourd’hui, beaucoup moins.

Le système commence à ressembler à :

- un pipeline de transformation documentaire,
- un moteur de publication,
- un système de synchronisation multi-supports,
- une couche de structuration des connaissances,
- et potentiellement une base exploitable par des systèmes IA.

Sans jamais avoir cherché à construire un CMS.

C’est presque l’inverse :

on garde volontairement :

- Markdown,
- Git,
- MkDocs,
- des fichiers simples,
- une architecture lisible.

Mais on ajoute progressivement une couche d’intelligence au-dessus.

---

## Une des leçons les plus intéressantes

Je crois qu’un des enseignements majeurs de ce projet est celui-ci :

> la documentation n’est pas seulement du contenu.

C’est une structure.

Un graphe.

Une représentation du fonctionnement réel d’un système.

Et à partir du moment où cette structure est propre :

- on peut la publier,
- la transformer,
- la relier,
- la requêter,
- la visualiser,
- et demain probablement raisonner dessus avec des agents IA.

La frontière entre “documentation”, “knowledge graph” et “système d’assistance intelligent” commence alors à devenir étonnamment floue.

Et c’est exactement ce genre de zone grise technique que j’adore explorer.
