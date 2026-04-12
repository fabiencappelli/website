---
title: "Architecture of v2"
date: 2026-04-04
lang: en
project: robie
image:
categories: [robie]
summary: "An attempt to visualize what v2 could look like"
---

While trying to think through the behavior I really want, I realized my naïve approach was incomplete. Robie will actually need to **listen while it is reading**. Because we’ll want to interrupt it:

- to change the story
- to switch to another action
- to adjust the volume, since right now there is nothing in place to control Robie’s volume

To kick off the thinking process, nothing beats a small diagram.

## Flow

<div class="mermaid">

flowchart TB
Start([Start]) --> Idle[Idle: waiting for wake word]

    Idle --> Wake[/Wake word said/]
    Wake --> Listen[/Record or listen live/]
    Listen --> DetectIntent[Process intent]
    DetectIntent --> ConfirmIntent[/Confirm intent/]
    ConfirmIntent --> IsConfirmIntent{Intent confirmed?}

    IsConfirmIntent -- No --> Listen
    IsConfirmIntent -- Yes --> IsIntent{Which intent?}

    IsIntent -- Read a story --> ReadingInit[Enter reading mode]
    IsIntent -- Take a note --> NotePrompt[/Please record the note/]
    IsIntent -- Other request --> HandleOther[Handle other intent]

    NotePrompt --> NoteListen[/Record note/]
    NoteListen --> NoteSave[Save note]
    NoteSave --> Idle

    HandleOther --> Idle

    subgraph ReadingMode [Reading mode]
        ReadingInit --> StartPlayback[Start audio playback]
        StartPlayback --> ReadingLoop[Reading active]

        ReadingLoop --> CheckCommand{Command detected?}
        ReadingLoop --> CheckTime{Is it midnight?}
        ReadingLoop --> EndOfStory{Story finished?}

        CheckCommand -- No --> ReadingLoop
        CheckCommand -- Stop --> StopPlayback[Stop playback]
        CheckCommand -- Volume up --> VolumeUp[Increase volume]
        CheckCommand -- Volume down --> VolumeDown[Decrease volume]

        VolumeUp --> ReadingLoop
        VolumeDown --> ReadingLoop

        CheckTime -- No --> ReadingLoop
        CheckTime -- Yes --> Shutdown[Shutdown device]

        EndOfStory -- No --> ReadingLoop
        EndOfStory -- Yes --> ExitReading[Exit reading mode]
    end

    StopPlayback --> Idle
    ExitReading --> Idle

</div>

## Consequences

The central point is that **Reading is not a one-shot action**.  
It is a **long-running active mode**, during which several things must exist at the same time:

- continuous audio playback
- listening for control commands
- monitoring the time
- the ability to interrupt playback cleanly

In other words, my system can no longer be designed as a simple linear chain such as:

```text
wake → listen → STT → action → end
```

It must become a system with **persistent activity + concurrent events**.

### First Constraint: Concurrency

Since I do not want to split playback into tiny chunks, the reading must continue **while something else is happening**.

That implies some form of concurrency, typically:

- multithreading
- separate processes
- or a more advanced event loop

In all cases, we move beyond the logic of “one loop doing everything in order”.

Concretely, I will probably need at least:

- one component managing playback
- one component listening to the microphone
- one component processing commands
- one component monitoring the clock
- one orchestrator deciding what to do

## Second Constraint: Clean Inter-Component Communication

As soon as multiple activities run in parallel, I need to define how they communicate.

For example:

- the microphone module detects “stop”
- it must notify the playback module
- the clock module detects midnight
- it must trigger a global shutdown
- the playback module reaches the end of the file
- it must notify the system to return to `Idle`

So I can no longer rely on simple functions calling one another directly.
I need logic such as:

- events
- message queues
- state flags
- synchronization objects

Otherwise, I’ll quickly end up with spaghetti code.

## Third Constraint: A Real State Model

My diagram implicitly says that we are no longer only in “do an action”, but in “be in a state”.

For example:

- `Idle`
- `Listening`
- `Reading`
- `Note recording`
- maybe later `Thinking`
- maybe `Shutting down`

And while in `Reading`, some commands are allowed:

- stop
- volume up
- volume down

while others may not be allowed, or not handled the same way.

So I need to explicitly model:

- the current state
- allowed transitions
- what happens when an event arrives in a given state

Otherwise I’ll get fuzzy behaviors like:

“What does Robie do if someone talks while it is reading?”
“What happens if midnight occurs during a volume command?”

## Fourth Constraint: Clean Interruption

Continuous audio playback means I must be able to:

- stop immediately
- possibly pause
- change volume on the fly
- exit without leaving the audio system in a broken state

So the audio player cannot be a simple blocking command launched without control.
It must be a controllable component, with commands such as:

- start
- stop
- pause
- set_volume

And those commands must remain safe no matter when they arrive.

## Fifth Constraint: Speech Recognition Can No Longer Be Designed the Same Way

In a classic conversational loop, we do:

- record
- transcribe
- act

But here, during reading, I need to detect **very short commands continuously**.

So I am no longer doing only “classic” STT.
Instead, I need continuous control listening, probably with:

- reduced vocabulary
- limited command logic
- fast and robust detection

So the problem is no longer:

“transcribe an open request”

but rather:

“quickly and reliably detect a few critical commands”

That is a different kind of need.

## Sixth Constraint: Risk of Robie Hearing Itself

This is probably one of the hidden big challenges of reading mode.

If Robie reads aloud while listening, the microphone may capture:

- its own playback
- reverberation
- children’s voices
- ambient noise

So I’ll need safeguards such as:

- short and highly specific commands
- adapted thresholds / detection logic
- maybe a secondary wake word in reading mode
- or microphone / volume / physical placement adjustments

The diagram does not mention it, but formalizing `Reading` as an interactive mode directly creates this problem.

## Seventh Constraint: Priority Logic

Not all events have the same weight.

For example:

- `Shutdown at midnight` is probably highest priority
- `Stop playback` is very high priority
- `Volume up` is less critical
- `Story finished` is a normal event

So I will need to define a policy:

- what interrupts what
- who wins in case of collision
- in what order events are processed

Without that, strange behaviors are likely.

## Eighth Constraint: Separate Behavior from Implementation

The diagram is excellent because it formalizes the expected behavior.
But it also forces an important distinction:

- **functional level**: what Robie must do
- **technical level**: how it is implemented

In this case, the formalization already says I will probably need:

- a controllable audio player
- parallel listening
- autonomous time monitoring
- an event system
- explicit state management

Even if I have not yet chosen between:

- thread
- callback
- queue
- process

## Summary

This formalization leads to one clear conclusion:

Robie can no longer be developed as a simple sequential pipeline.
Reading mode requires a **concurrent, event-driven architecture with explicit state management**.

More concretely, that means:

- several activities must run at the same time
- they must communicate cleanly
- the system must know its current state
- playback must be interruptible at any moment
- voice detection during playback becomes a specific problem
- priorities and transitions must be handled properly
