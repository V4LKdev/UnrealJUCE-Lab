# UnrealJUCE-Lab - UltraLowLatencyAudio (UE5 C++ Plugin + JUCE)

A small Unreal Engine 5 plugin lab that integrates **JUCE** as a custom audio/MIDI backend to explore **ultra-low-latency device I/O** (primarily via **ASIO on Windows**).  
This is a technical code sample / experiment, not in any way a production-ready usable replacement for Unreal’s audio pipeline.

- **Stack:** Unreal Engine 5 plugin (C++), JUCE static library (external), CMake/Projucer build for JUCE
- **Focus:** real-time audio callbacks, buffer negotiation, MIDI event handling, clean ThirdParty linkage

**Related music-tech demos (YouTube):**  
These clips show other parts of my broader UE/JUCE music framework that aren’t in this repo.

- **[Quartz-synchronized MetaSounds music system demo](https://www.youtube.com/watch?v=uktz1KixA6s)** – beat-accurate playback + gameplay-driven transitions.  
- **[Real-time MIDI input and Sample demo](https://www.youtube.com/watch?v=6hI5Qhj0GSw)** – low-latency MIDI capture and instrument routing.

---

## Current status / known issues

- The JUCE device callback path is working and can achieve very low latency with ASIO.
- The Unreal integration layer and module linking are stable for Win64.

**Known issue (important):**
- When running audio through this backend over asio, Unreal’s native audio manager / sound class routing is currently disrupted.  
  In other words, ACO/JUCE output does not participate cleanly in Unreal’s normal SoundClass/SoundMix/AudioMixer control.  
  A proper solution will likely require a controlled bridge back into UE’s mixer or a hybrid routing approach.

---

## Key files

- **JUCE linkage & platform config:** `UltraLowLatencyAudio.Build.cs`  
- **Unreal ↔ JUCE boundary / includes:** `Private/JUCEPCH.h`  
- **Audio device + callback core:** `Private/Audio/AudioEngine.*`  
- **MIDI handling:** `Private/Audio/MidiCallback.*`, `Private/MIDIThread/*`  
- **Unreal integration entrypoint:** `Private/Subsystem/*`

---

## Third-Party JUCE setup

JUCE is **not included** in this repo. Build it separately and place it in `ThirdParty/`.

### 1) Obtain JUCE
Download and unpack JUCE from the official site:  
https://juce.com

### 2) Build a JUCE static library
Using **Projucer**, create a **static library** containing the JUCE modules you need.

Modules expected by this plugin (see `JUCEPCH.h`):
- `juce_core`
- `juce_events`
- `juce_audio_basics`
- `juce_audio_devices`
- `juce_audio_processors`
- `juce_audio_utils`

### 3) Place headers + library here
Expected layout:

```ThirdParty/
JUCE/
Includes/
modules/ <-- JUCE module headers
Libraries/
Release/
JUCEStaticLib.lib <-- your built static lib (Win64)
```

If your library name/path differs, update `UltraLowLatencyAudio.Build.cs` accordingly.

### ASIO (Windows)
To use ASIO:
1. Install ASIO drivers on your system.
2. In Projucer, enable ASIO support in `juce_audio_devices` **before** building the static library.

---

## Build notes

- RTTI and exceptions are enabled in `Build.cs` to match JUCE requirements.
- `WITH_JUCE_BINDING` is used to guard compilation when JUCE is not present.

---

## Usage (lab workflow)

1. Add the plugin to a UE5 project.
2. Ensure JUCE is linked (setup above).
3. Launch editor / PIE.
4. The subsystem initializes JUCE audio + MIDI, and `AudioEngine` drives the callback/render path.
5. Use the Blueprint exposed functions from the subsystem to interact with JUCE
