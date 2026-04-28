---
title: Release Video Generator
slug: release-video-generator
summary: Pipeline Python qui transforme un export Jira HTML en vidéo de release narrée, avec génération de script thématique, synthèse vocale et rendu vidéo automatisé.
stack: [Python, OpenAI API, BeautifulSoup, FFmpeg, Pydub]
featured: true
order: 3
github_repos:
  - url: https://github.com/fabiencappelli/releaser
    label: GitHub
---

## Générateur de vidéo de release à partir de tickets Jira

Ce projet automatise la création d’une vidéo de mise à jour produit à partir d’un export HTML Jira.

L’idée est de convertir des tickets techniques en un format narratif court et lisible, grâce à une chaîne complète en 5 étapes :

- parsing des tickets Jira,
- regroupement thématique et génération du script,
- synthèse vocale de chaque segment,
- normalisation/assemblage audio,
- rendu des slides et export MP4 final.

## Contexte & Objectifs

### Contexte

Les notes de release sont souvent trop longues, peu homogènes et difficiles à diffuser auprès d’équipes non techniques.

Ce projet répond à ce besoin en transformant automatiquement un export Jira en vidéo narrée, plus simple à consommer pour :

- les équipes produit,
- les développeurs,
- les parties prenantes internes,
- et pourquoi pas, à terme, les utilisateurs finaux.

### Objectifs

- Structurer automatiquement des tickets Jira en thèmes cohérents.
- Générer un script de présentation clair, précis et exploitable à l’écran + en voix off.
- Produire une narration audio segmentée puis synchronisée avec les slides.
- Générer une vidéo finale standardisée (`output/release.mp4`) sans montage manuel.

## Vue d’ensemble du pipeline

<div class="mermaid">
flowchart TD
    A["Export Jira (HTML)"] --> B["Parse tickets (BeautifulSoup)"]
    B --> C["Groupement thématique + script (OpenAI)"]
    C --> D["Génération TTS par segment (OpenAI)"]
    D --> E["Normalisation WAV + pauses + concat audio (FFmpeg)"]
    E --> F["Rendu slides + concat vidéo + mux audio/vidéo (FFmpeg)"]
    F --> G["release.mp4"]
</div>

## Données d’entrée et artefacts

| Élément             | Description                     |
| :------------------ | :------------------------------ |
| Entrée principale   | `data/jira_export.html`         |
| Données structurées | `build/release_structured.json` |
| Thèmes générés      | `build/themes.json`             |
| Script vidéo        | `build/script.json`             |
| Audios segmentés    | `audio/*.mp3`                   |
| Audio final         | `build/final_audio.wav`         |
| Timeline            | `build/timeline.json`           |
| Vidéo finale        | `output/release.mp4`            |

## Détail des composants

### 1) Parsing Jira (`src/parse_jira.py`)

- Lit l’export HTML Jira avec BeautifulSoup.
- Extrait les colonnes clés (type, key, summary, created, assignee, reporter, description).
- Filtre actuellement les tickets par préfixe de clé (`OPD-`).
- Exporte un JSON structuré exploitable par les étapes suivantes.

### 2) Génération des thèmes et du script (`src/generate_script.py`)

Le module réalise 2 passes via l’API OpenAI :

- **Pass 1** : regrouper les tickets en thèmes fonctionnels (classification `UI`, `API` ou `unspecified`).
- **Pass 2** : générer les slides finales (intro, slides thématiques, conclusion) avec :
  - un titre,
  - 3 à 5 bullets à l’écran,
  - une voix off de 3 à 5 phrases.

Le ton demandé au modèle est technique, clair et non marketing.

### 3) Génération audio (`src/generate_audio.py`)

- Lit `build/script.json`.
- Génère un fichier MP3 par segment via `gpt-4o-mini-tts` (voix `alloy`).
- Nomme les fichiers de manière ordonnée (`00_intro.mp3`, etc.).

### 4) Construction de la piste audio finale (`src/build_audio_track.py`)

- Convertit les MP3 en WAV normalisés (48kHz, stéréo, PCM s16le).
- Insère des pauses silencieuses entre les segments.
- Concatène le tout dans `build/final_audio.wav`.

### 5) Rendu vidéo (`src/render_video.py`)

- Génère une slide vidéo silencieuse par segment en superposant texte + panneaux sur un background.
- Ajuste la durée de chaque slide sur la durée audio réelle (via `ffprobe`).
- Insère des segments visuels de pause.
- Concatène les segments vidéo, puis mux la piste audio finale.
- Produit `output/release.mp4` et une timeline JSON.

### Orchestration complète (`run_all.py`)

Le script `run_all.py` enchaîne automatiquement les 5 étapes avec gestion d’erreurs et logs par phase.

## Stack technique

- **Langage** : Python 3.10+
- **IA (LLM + TTS)** : OpenAI API
- **Parsing HTML** : BeautifulSoup4
- **Audio** : Pydub + FFmpeg
- **Vidéo** : FFmpeg (drawtext, concat, mux)
- **Config** : `python-dotenv` pour la clé API

## Points forts

- Pipeline de bout en bout, automatisé et reproductible.
- Bonne séparation des responsabilités (scripts dédiés par étape).
- Structure d’artefacts claire (`build/`, `audio/`, `output/`).
- Synchronisation audio/vidéo robuste grâce à la normalisation WAV et au probe des durées.

## Limites actuelles

- Dépendance forte à la qualité de l’export Jira HTML.
- Layout visuel volontairement simple (fond statique, pas d’animations avancées).
- Modèles et paramètres OpenAI codés en dur (peu de configuration externe).
- Filtrage des tickets actuellement spécifique au préfixe `OPD-`.

## Perspectives d’évolution

- Paramétrage via fichier de config (modèles, voix, style des slides, filtres Jira).
- Multiples templates visuels et thèmes de rendu.
- Ajout de sous-titres automatiques.
- Internationalisation (langue de narration configurable).
- Packaging CLI pour un usage “one command” en CI/CD release.
