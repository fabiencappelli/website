---
title: "Continuation de la v1"
date: 2026-04-03
lang: en
project: robie
image: /assets/images/blog/robie_1.jpg
categories: [robie, audio]
summary: "Reconstitution d'un environnement virtuel sain, travail sur la transcription"
---

ANGLAIS
![image](/assets/images/blog/robie_1.jpg)

Voici un article structuré, clair et fidèle à ce que tu as vécu — sans bullshit, avec une vraie narration technique.

---

# 🤖 Reconstitution d'un environnement virtuel sain

Et... patatra...
Tout se brise, et notamment la gestion du bonnet voice Adafruit.

Je dois tout recommencer pour faire en sorte que le son entre et sorte.

---

## 🔌 Étape 1 — Le vrai mur : l’audio bas niveau

Avant même de parler d’IA, le premier défi a été… le micro.

### Problèmes rencontrés :

- erreurs `RPi.GPIO` → conflit entre environnement Python et libs système
- `sounddevice` incapable d’ouvrir le flux audio
- PulseAudio / PipeWire qui monopolisent le device
- ALSA qui voit la carte… mais refuse tous les formats

### Symptômes typiques :

- `PortAudioError: Invalid number of channels`
- `device or resource busy`
- `Unable to install hw params`

### Leçons importantes :

- Sur Raspberry Pi, **éviter les couches audio haut niveau**
- Aller directement vers **ALSA (`arecord`)**
- Désactiver PipeWire/PulseAudio si nécessaire
- Vérifier la config du codec via `alsamixer`

👉 Une fois cette étape passée, tout devient beaucoup plus simple.

---

## 🎧 Étape 2 — Pipeline audio fonctionnel

Après stabilisation, on obtient enfin :

```text
micro → ALSA → enregistrement → traitement → playback
```

Et côté UX :

- LED éteinte → veille
- LED rouge → écoute
- LED jaune → traitement
- son → réponse

👉 À ce stade, le robot “vit” déjà.

---

## 🧠 Étape 3 — Tentative avec Whisper (et échec)

L’étape suivante logique était la transcription avec Whisper.

### Résultat :

- latence énorme (plusieurs secondes voire dizaines de secondes)
- mauvaise qualité avec modèle `tiny`
- impossible de monter en qualité sans exploser le temps de calcul

### Pourquoi ça échoue :

- Raspberry Pi 4 trop limité pour du STT moderne
- Whisper optimisé pour GPU ou CPU puissants
- compromis qualité / vitesse impossible à tenir

👉 Conclusion :
**Whisper est excellent… mais pas pour ce cas d’usage sur Pi.**

---

## 🔄 Étape 4 — Pivot vers Vosk

Changement de stratégie : tester Vosk.

### Résultat immédiat :

- latence bien meilleure
- transcription presque correcte
- pipeline stable

👉 Grosse amélioration.

Mais…

### Nouveau problème :

- ~10 secondes pour traiter 4 secondes d’audio
- encore trop lent pour une interaction naturelle

---

## 💡 Compréhension clé : mauvais problème

Le problème n’était pas le moteur.

👉 Le problème était la tâche.

On demandait :

> “Transcris librement tout ce que je dis”

Alors que le vrai besoin était :

> “Reconnais quelques commandes simples”

---

## 🎯 Étape 5 — Changement de paradigme

Au lieu de faire de la dictée vocale, on passe à :

👉 **reconnaissance de commandes vocales**

### Exemple :

```python
if "bonjour" in text:
    play("bonjour.mp3")
```

Ou mieux encore : limiter le vocabulaire directement dans Vosk :

```python
rec = KaldiRecognizer(
    model,
    16000,
    '["bonjour", "histoire", "musique", "stop"]'
)
```

### Résultat :

- plus rapide
- plus fiable
- beaucoup plus robuste

---

## 🧠 Architecture finale (V1)

```text
Wake word
    ↓
LED rouge (écoute)
    ↓
Enregistrement court (2–3s)
    ↓
Vosk (vocabulaire limité)
    ↓
Intent simple
    ↓
Réponse audio
    ↓
Retour veille
```

---

## ⚡ Ce qui a vraiment fait la différence

### ❌ Ce qui ne marche pas bien

- Whisper sur Raspberry Pi
- audio abstrait (sounddevice, PulseAudio)
- transcription libre sur CPU faible

### ✅ Ce qui marche

- ALSA direct (`arecord`)
- pipeline simple et déterministe
- Vosk avec vocabulaire restreint
- logique d’intentions plutôt que NLP complet

---

## 🚀 Résultat

On passe de :

> un prototype lent et frustrant

à :

> un assistant vocal rapide, réactif et utilisable par des enfants

---

## 🔮 Et après ?

Une fois cette base solide :

- ajouter détection de fin de parole (VAD)
- améliorer les réponses (TTS ou sons)
- ajouter mémoire simple
- éventuellement brancher un LLM (mais plus tard)

---

## 🧭 Conclusion

Le point clé de ce projet n’a pas été un problème d’IA.

C’était un problème de **choix d’architecture**.

👉 Sur du matériel limité :

- il faut **simplifier le problème**
- pas juste optimiser la solution

---

Si tu construis un assistant embarqué :

> commence simple, local, rapide
> puis complexifie seulement quand l’expérience utilisateur est déjà bonne

---

Et honnêtement :

👉 faire dire “bonjour” à un robot en moins d’une seconde
vaut largement mieux que comprendre Shakespeare en 10 secondes.
