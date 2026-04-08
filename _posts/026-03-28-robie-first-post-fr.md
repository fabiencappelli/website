---
title: "Démarrage du projet"
date: 2026-03-28
lang: fr
project: robie
image: /assets/images/blog/robie_1.jpg
categories: [robie, audio]
summary: "Premiers essais avec openWakeWord, LEDs et logique d’écoute sur Raspberry Pi."
---

![image](/assets/images/blog/robie_1.jpg)

# 🎯 Objectif

Construire un système capable de :

- écouter en permanence
- détecter un mot d’éveil (wake word)
- enregistrer une commande vocale
- comprendre l’intention
- déclencher une action
- répondre avec du son ou de la voix

Le tout **localement**, sans dépendance cloud.

---

# 🧱 Architecture globale

Le système se décompose en plusieurs briques :

```text
Micro
→ Wake word
→ Enregistrement
→ Speech-to-Text (Whisper)
→ Interprétation (LLM)
→ Action
→ Réponse (sons / TTS / LEDs)
```

Chaque brique a été explorée et validée séparément.

---

# 🎧 1. Audio et matériel

Le projet s’appuie sur :

- Raspberry Pi (Debian Bookworm)
- Adafruit Voice Bonnet (micros + LEDs + haut-parleurs)
- sortie audio testée via bruit rose

Points importants :

- bien identifier le bon device audio
- gérer correctement l’entrée/sortie simultanée
- prévoir une gestion propre des LEDs (cleanup)

---

# 🔊 2. Wake word : le premier défi

## ❌ Picovoice (Porcupine)

Initialement envisagé, mais abandonné :

- demande désormais un compte pro
- dépendance externe
- moins adapté à un projet perso long terme

## ✅ openWakeWord

Choix retenu :

- open source
- fonctionne en local
- basé sur des modèles TFLite

### Difficultés rencontrées :

- modèles manquants → `download_models()`
- conflits NumPy / SciPy → downgrade et alignement versions
- faux positifs → nécessité de filtrage

### Solutions mises en place :

- seuil élevé (`~0.95`)
- plusieurs frames consécutives
- période réfractaire (10s)
- arrêt du stream pendant l’action

👉 Le point clé : **un wake word n’est pas fiable “brut” — il faut le cadrer**

---

# 🤖 3. Problème classique : le robot s’auto-déclenche

Robie détectait son propre son (feedback audio).

Solution :

- couper l’écoute pendant :
  - enregistrement
  - réponse sonore

- ajouter un délai de grâce

👉 Sans ça, boucle infinie garantie.

---

# 🧠 4. STT ≠ compréhension

Whisper (ou whisper.cpp) fait :

👉 **audio → texte**

Mais ne comprend rien.

La “compréhension” est une autre couche.

---

# 🤖 5. Introduction d’un LLM local

Test avec Ollama + Qwen2.5 (1.5B)

Résultat :

- latence ~2 secondes
- fonctionnement stable
- viable pour un usage embarqué

👉 Conclusion : **un petit LLM local sur Pi est exploitable**

---

# ⏱️ La métrique clé : la latence

Ce qui compte n’est pas la vitesse globale, mais :

👉 **le temps avant que la réponse commence**

- < 3 secondes → bon
- 3–6 → acceptable
- > 6 → frustrant

Astuce UX :

- LED jaune = “réflexion”
- son intermédiaire

👉 transforme un lag en comportement naturel

---

# 🧠 6. Rôle du LLM dans Robie

Le LLM n’est pas utilisé pour “discuter”.

👉 Il sert à :

- transformer une phrase en intention
- structurer la commande

Exemple :

Entrée :

> “Robie, note que Thomas doit prendre son manteau demain”

Sortie :

```json
{
  "intent": "take_note",
  "content": "Thomas doit prendre son manteau demain",
  "answer": "C’est noté."
}
```

👉 Le code Python exécute ensuite l’action.

---

# 🌍 7. Support du français

Contrainte importante : enfants francophones.

Solutions :

- Whisper multilingue (pas `.en`)
- langue forcée en `fr`
- LLM multilingue (Qwen OK)
- prompts en français
- intents en français

⚠️ Attention : la reconnaissance des voix d’enfants est plus difficile → prévoir tolérance.

---

# ⚡ 8. Et la Coral TPU ?

Non utilisable pour les LLM.

Pourquoi :

- Coral = modèles TFLite quantifiés
- LLM = architecture incompatible

👉 Usage futur pertinent :

- vision (caméra)
- détection d’objets
- perception environnement

---

# 🤖 9. Et le Jetson ?

Perspective long terme.

## Raspberry Pi

- bon pour :
  - audio
  - logique
  - petits LLM

## Jetson Orin

- permet :
  - LLM plus gros (3B–7B)
  - vision temps réel
  - multimodal

👉 Pi = prototype
👉 Jetson = robot complet

---

# 🔌 10. Détail matériel : la LED rouge

Après `shutdown`, la LED rouge reste allumée.

👉 normal :

- LED = alimentation
- Pi n’a pas de bouton OFF

👉 seule solution : couper le courant

---

# 🚀 Conclusion

Ce projet montre qu’il est possible de construire un assistant embarqué local :

## Ce qui fonctionne aujourd’hui

- wake word local (avec tuning)
- enregistrement audio stable
- LLM local utilisable (~2s)
- pipeline complet fonctionnel

## Ce qui reste à améliorer

- wake word personnalisé (“Hey Robie”)
- robustesse STT (enfants)
- TTS naturel
- gestion des intents plus riche

---

# 🧠 Insight clé

Un assistant embarqué n’est pas :

👉 “un modèle puissant”

C’est :

👉 **une architecture bien découpée**

---

# 🔮 Suite du projet

- intégration Whisper → LLM → actions
- mémoire (notes, contexte)
- personnalité Robie
- vision (via Jetson ou Coral)

---

Robie V1 n’est pas encore “intelligent”.
Mais il est déjà **vivant**.

Et c’est là que tout commence.
