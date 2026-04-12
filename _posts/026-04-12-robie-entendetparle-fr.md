---
title: "Robie V1 — Wake Word, STT et TTS sur Raspberry Pi"
date: 2026-04-12
lang: fr
project: robie
categories: [robie, raspberry-pi, ia, audio]
image: /assets/images/blog/robie_2.png
summary: "Première boucle conversationnelle locale pour Robie : détection du wake word, transcription vocale en français et réponse parlée sur Raspberry Pi."
---

## Une vraie boucle vocale locale

La session du jour avait un objectif simple : transformer Robie en quelque chose de plus qu’un prototype qui clignote.

L’idée était de valider une chaîne complète, embarquée sur Raspberry Pi :

- attente d’un mot-clé (_wake word_)
- écoute de la commande
- transcription vocale en français
- restitution de la phrase en synthèse vocale
- retour en veille

Autrement dit : une première boucle conversationnelle locale, sans dépendre d’un service cloud.

---

## Architecture testée

Le pipeline retenu repose sur des composants légers et adaptés au Raspberry Pi :

- **OpenWakeWord** pour la détection du mot d’activation
- **SoundDevice** pour la capture audio
- **Vosk** pour la reconnaissance vocale hors ligne
- **Pico TTS** pour la synthèse vocale
- **DotStar LEDs** pour le feedback visuel

Le comportement est volontairement simple :

- LED éteintes en veille
- rouge pendant l’écoute
- jaune pendant le traitement
- lecture de la réponse
- extinction et reprise de la veille

---

## Bonne surprise : Pico TTS

La découverte la plus positive de la journée a été la rapidité de **Pico TTS**.

La voix est clairement synthétique, presque rétro, mais la génération est immédiate et parfaitement exploitable sur une machine modeste. Dans le cadre de Robie, ce défaut devient presque une qualité : le rendu robotique colle bien à l’identité du projet.

Le vrai enjeu n’est donc pas tant la voix que la qualité de la chaîne audio (haut-parleurs, volume, mixage, sortie sonore).

---

## Résultats sur la reconnaissance vocale

Les tests ont confirmé que la transcription locale fonctionne, mais avec des limites prévisibles :

- latence perceptible
- résultats variables selon la clarté de la diction
- performances plus fragiles avec les voix d’enfants
- davantage de difficulté avec les plus jeunes utilisateurs

Un point intéressant est déjà apparu : l’outil fonctionne mieux quand la personne parle de manière nette et sans hésitation. Cela signifie qu’une partie de l’expérience utilisateur passera aussi par l’apprentissage du bon usage du système.

---

## Ce que cette session valide

Même imparfait, le prototype prouve plusieurs choses importantes :

- un assistant vocal local sur Raspberry Pi est réaliste
- des briques open source suffisent pour une V1 crédible
- la boucle complète wake word → STT → TTS fonctionne réellement
- les limites actuelles sont davantage ergonomiques que conceptuelles

C’est un cap plus important qu’il n’y paraît : Robie n’est plus seulement un assemblage de composants, mais un objet qui écoute, comprend parfois, et répond.

---

## Pistes d’amélioration

Les prochaines itérations pourront viser :

- meilleure reconnaissance des voix enfantines
- réduction de la latence
- amélioration de la sortie audio
- dialogues plus naturels
- interruption pendant la lecture
- gestion d’histoires, notes vocales et commandes multiples

---

## Conclusion

Le résultat n’est pas parfait. Robie est parfois lent, parfois hésitant, parfois maladroit.

Mais il fonctionne, et les enfants étaient surexcités.
