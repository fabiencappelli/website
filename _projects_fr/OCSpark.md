---
title: Projet Big Data Cloud — Pipeline PySpark sur AWS pour la reconnaissance d’images (Fruits-360) (Projet Open Classroom 11)
slug: OCSpark
summary: Conception et déploiement d’un pipeline Big Data scalable pour le traitement massif d’images, l’extraction automatique de caractéristiques visuelles et la réduction de dimension, dans une logique de workflow data engineering et machine learning orienté production.
stack: [Python, AWS S3, EMR, Spark, PySpark, JupyterHub, MobileNetV2, PCA]
featured: true
order: 6
---

## Contexte

L’idée de départ est une application de sensibilisation à la biodiversité basée sur la **reconnaissance de fruits**.
Le problème est typique d’un cas réel : un pipeline qui fonctionne sur une machine locale finit par ne plus tenir dès que le volume de données augmente.

Ce projet répond donc à une question simple :

> Comment passer d’un notebook “data science” classique à une exécution distribuée, scalable et maîtrisée en coûts, sur le cloud ?

---

## Jeu de données

J’ai utilisé le dataset **Fruits-360** (Mihai Oltean, 2017), et plus précisément le set de test.

- **22 819 images**
- **131 classes** (variétés de fruits : pommes, poires, etc.)
- Images RGB, résolution **100×100**
- Labels dérivés de l’arborescence des fichiers (noms de dossiers)

Ce dataset est un bon terrain d’entraînement, car il permet d’industrialiser un pipeline sans dépendre d’une API externe ou d’annotations manuelles.

---

## Architecture cible

Le pipeline repose sur une architecture cloud simple mais robuste :

- **S3** : Data Lake (images brutes + résultats)
- **EMR** : cluster Spark/Hadoop
- **IAM** : gestion des accès (et contraintes de conformité)
- **JupyterHub** : interface notebooks PySpark sur le Master Node

Le schéma général est le suivant :

<div class="mermaid">
flowchart TD
  A["S3 (images)"] --> B
  B["EMR Spark (feature extraction + PCA)"] --> C["S3 (features réduites)"]
</div>

---

## Mise en place de l’environnement AWS

La mise en place a inclus toutes les étapes classiques d’un déploiement :

1. Création d’un bucket S3 et import des images
2. Configuration IAM (rôles + politiques, approche “least privilege”)
3. Lancement d’un cluster EMR : **1 Master + 2 Workers**
4. Mise en place d’un tunnel proxy pour accéder à JupyterHub
5. Connexion JupyterHub
6. Exécution du notebook PySpark
7. Sauvegarde des résultats sur S3

---

## Traitement distribué PySpark : pipeline complet

### 1) Lecture des images dans Spark

Les images sont chargées via le format Spark `binaryFile`, ce qui permet de manipuler le dataset comme un DataFrame distribué.

- Lecture récursive des fichiers
- Création d’une colonne label depuis le chemin (`path`)
- Partitionnement pour le parallélisme

---

### 2) Feature extraction avec MobileNetV2 (poids ImageNet)

L’étape centrale consiste à transformer chaque image en vecteur de caractéristiques (embedding).

J’ai utilisé :

- **MobileNetV2 pré-entraîné ImageNet**
- suppression de la couche de classification (on garde la sortie de la couche finale convolutionnelle)
- modèle en mode inference (poids gelés)

Le point important ici est que l’on ne “réentraîne” pas un modèle : on l’utilise comme **extracteur de features**, ce qui est typique en production lorsque l’on veut :

- standardiser une représentation
- réduire les coûts
- éviter de relancer des entraînements lourds

---

### 3) Broadcast des poids (optimisation Spark)

Sur un cluster Spark, charger un modèle deep learning sur chaque worker peut exploser la mémoire et le temps d’exécution.

Pour éviter ça, j’ai utilisé une stratégie Spark classique :

- **broadcast des poids**
- reconstruction du modèle sur chaque worker en réutilisant les poids partagés

Ce point est intéressant car il montre une compréhension de l’exécution distribuée : on ne fait pas “juste tourner un notebook sur EMR”, on optimise réellement la distribution.

---

### 4) Réduction de dimension (StandardScaler + PCA)

Les embeddings extraits sont ensuite standardisés puis réduits via PCA.

Pipeline Spark ML :

- `StandardScaler`
- `PCA`

Objectif :

- réduire la taille des données
- faciliter la visualisation
- préparer un futur modèle (classification, clustering, recherche de similarité)

---

### 5) Sauvegarde des résultats sur S3

Le résultat final (features réduites + labels + path) est sauvegardé sur S3 au format **Parquet**, ce qui est un standard en environnement Big Data.

---

## Maîtrise des coûts (cloud “réaliste”)

J’ai volontairement intégré un objectif de maîtrise budgétaire, comme on le ferait dans un contexte entreprise.

- Région : **eu-west-1 (Irlande)**
- Extinction automatique après tests
- Coût global : **< 10 €**

📌 Ce point est important : beaucoup de projets cloud échouent en pratique non pas techniquement, mais parce que la facture explose.

---

## Résultats obtenus

Ce projet produit une sortie directement exploitable :

- **22 819 images traitées**
- **131 classes**
- Embeddings extraits via MobileNetV2
- Dataset final réduit via PCA
- Résultats stockés sur S3 en Parquet

---

## Conclusion

Ce projet m’a permis de pratiquer une chaîne complète **Cloud + Big Data + ML**, depuis la collecte et le stockage jusqu’à la transformation distribuée et la préparation de données pour un modèle.

C’est typiquement le type de pipeline que l’on retrouve dans des cas réels : vision par ordinateur, catalogues produits, contrôle qualité, ou recherche d’images similaires, avec un besoin clair de **scalabilité**.
