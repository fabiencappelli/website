---
title: "Continuation of v1"
date: 2026-04-03
lang: en
project: robie
image: /assets/images/blog/robie_1.jpg
categories: [robie, audio, test2]
summary: "Rebuilding a clean virtual environment, working on transcription"
---

![image](/assets/images/blog/robie_1.jpg)

---

## Rebuilding a Clean Virtual Environment

And... crash.

Everything broke, especially the Adafruit Voice Bonnet handling.

I had to start over to make audio input and output work again.

---

### Step 1 — The Real Wall: Low-Level Audio

Before even talking about AI, the first challenge was… the microphone.

#### Problems encountered:

- `RPi.GPIO` errors → conflict between Python environment and system libraries
- `sounddevice` unable to open the audio stream
- PulseAudio / PipeWire locking the device
- ALSA detects the card… but rejects every format

#### Typical symptoms:

- `PortAudioError: Invalid number of channels`
- `device or resource busy`
- `Unable to install hw params`

#### Important lessons:

- On Raspberry Pi, **avoid high-level audio layers**
- Go directly through **ALSA (`arecord`)**
- Disable PipeWire/PulseAudio if needed
- Check codec configuration with `alsamixer`

Once this step is solved, everything becomes much easier.

---

### Step 2 — Working Audio Pipeline

After stabilization, we finally get:

```text
microphone → ALSA → recording → processing → playback
```

And on the UX side:

- LED off → standby
- red LED → listening
- yellow LED → processing
- sound → response

At this stage, the robot already feels “alive”.

---

### Step 3 — Whisper Attempt (and Failure)

The next logical step was transcription with `faster-whisper`.

#### Result:

- huge latency (several seconds, sometimes tens of seconds)
- poor quality with the `tiny` model
- impossible to improve quality without exploding compute time

#### Why it fails:

- Raspberry Pi 4 is too limited for modern STT
- Whisper is optimized for GPUs or powerful CPUs
- impossible to maintain a good quality/speed tradeoff

Conclusion:
**Whisper is excellent… but not for this use case on Pi.**

---

### Step 4 — Pivot to Vosk

Strategy shift: test Vosk.

#### Immediate result:

- much better latency
- almost correct transcription
- stable pipeline

Big improvement.

But…

#### New problem:

- ~10 seconds to process 4 seconds of audio
- still too slow for natural interaction

---

#### Key Insight: Wrong Problem

The issue was not the engine.

The issue was the task.

We were asking:

> “Freely transcribe everything I say”

When the real need was:

> “Recognize a few simple commands”

---

### Step 5 — Paradigm Shift

Instead of voice dictation, move to **voice command recognition**.

#### Example:

```python id="a1r7kp"
if "hello" in text:
    play("hello.mp3")
```

Or even better: restrict the vocabulary directly in Vosk:

```python id="w2m5dz"
rec = KaldiRecognizer(
    model,
    16000,
    '["hello", "story", "music", "stop"]'
)
```

#### Result:

- faster
- more reliable
- much more robust

---

### Final Architecture (V1)

```text
Wake word
    ↓
Red LED (listening)
    ↓
Short recording (2–3s)
    ↓
Vosk (limited vocabulary)
    ↓
Simple intent
    ↓
Audio response
    ↓
Back to standby
```

---

### What Really Made the Difference

#### What does not work well

- Whisper on Raspberry Pi
- abstracted audio layers (`sounddevice`, PulseAudio)
- free transcription on a weak CPU

#### What works

- direct ALSA (`arecord`)
- simple and deterministic pipeline
- Vosk with restricted vocabulary
- intent logic rather than full NLP

---

### Result

We move from:

> a slow and frustrating prototype

to:

> a fast, responsive voice assistant usable by children

---

### What Comes Next?

Once this solid base is ready:

- add end-of-speech detection (VAD)
- improve responses (TTS or sounds)
- add simple memory
- possibly connect an LLM (later)

---

## Conclusion

The key issue in this project was not an AI problem.

It was an **architecture choice** problem.

On limited hardware:

- you must **simplify the problem**
- not just optimize the solution

---
