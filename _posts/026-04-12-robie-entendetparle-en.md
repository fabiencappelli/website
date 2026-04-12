---
title: "Robie V1 — Wake Word, STT and TTS on Raspberry Pi"
date: 2026-04-12
lang: en
project: robie
categories: [robie, raspberry-pi, ai, audio]
image: /assets/images/blog/robie_2.png
summary: "Robie’s first local conversational loop: wake word detection, French speech transcription, and spoken response on Raspberry Pi."
---

## A Real Local Voice Loop

Today's session had a simple goal: turn Robie into something more than a blinking prototype.

The idea was to validate a complete voice pipeline running locally on a Raspberry Pi:

- waiting for a wake word
- listening to the command
- speech transcription in French
- spoken playback using text-to-speech
- returning to idle mode

In other words: a first fully local conversational loop, without relying on any cloud service.

---

## Tested Architecture

The selected pipeline relies on lightweight components suitable for a Raspberry Pi:

- **OpenWakeWord** for wake word detection
- **SoundDevice** for audio capture
- **Vosk** for offline speech recognition
- **Pico TTS** for speech synthesis
- **DotStar LEDs** for visual feedback

The behavior is intentionally simple:

- LEDs off while idle
- red while listening
- yellow while processing
- playback of the response
- lights off and return to standby

---

## Pleasant Surprise: Pico TTS

The most positive surprise of the day was the speed of **Pico TTS**.

The voice is clearly synthetic, almost retro, but generation is immediate and perfectly usable on modest hardware. In Robie’s case, that limitation almost becomes a strength: the robotic sound fits the project’s identity.

The real challenge is therefore not the voice itself, but the quality of the audio chain (speakers, volume, mixing, output quality).

---

## Speech Recognition Results

The tests confirmed that local transcription works, but with predictable limitations:

- noticeable latency
- variable results depending on speech clarity
- weaker performance with children’s voices
- more difficulty with younger users

One interesting lesson already emerged: the system performs better when the speaker talks clearly and without hesitation. That means part of the user experience will also involve learning how to interact with it effectively.

---

## What This Session Validates

Even imperfect, the prototype proves several important points:

- a local voice assistant on Raspberry Pi is realistic
- open-source building blocks are enough for a credible V1
- the full wake word → STT → TTS loop genuinely works
- current limitations are more ergonomic than conceptual

This is a bigger milestone than it may seem: Robie is no longer just an assembly of components, but an object that listens, sometimes understands, and responds.

---

## Improvement Paths

Future iterations could focus on:

- better recognition of children's voices
- lower latency
- improved audio output quality
- more natural dialogues
- interruption during playback
- handling stories, voice notes, and multiple commands

---

## Conclusion

The result is not perfect. Robie is sometimes slow, sometimes hesitant, sometimes clumsy.

But it works — and the children were wildly excited.
