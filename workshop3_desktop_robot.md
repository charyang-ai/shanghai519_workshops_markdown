# Reachy Mini Chat Workshop — From v1 to v8

A hands-on guide covering emotion-driven robotics, TTS integration, and ASR pipelines. Each section is designed for ~5-10 minutes.

---

## Prerequisites (do this once provide by AMD Remote Machine)

### Install linux oem kernel and ROCm

The workshop is based on AMD Strix Halo or Strix Point with Ubuntu24.04 + ROCm preinstalled images.
This base images are setup by follow [Install Ryzen Software for Linux with ROCm](https://rocm.docs.amd.com/projects/radeon-ryzen/en/latest/docs/install/installryz/native_linux/install-ryzen.html)
 
The commands are copied here as reference. 
NOTE: Suppose it has been done by AMD Remote Machine, Developer could skip this part

```
# Install the oem kernel
sudo apt update && sudo apt install linux-oem-24.04c
sudo reboot
```

```
# Install rocm
uname -r
sudo apt upgrade -y
sudo apt update
wget https://repo.radeon.com/amdgpu-install/7.2.1/ubuntu/noble/amdgpu-install_7.2.1.70201-1_all.deb
sudo apt install ./amdgpu-install_7.2.1.70201-1_all.deb

amdgpu-install -y --usecase=rocm --no-dkms

# Set groups permissions
groups
sudo usermod -a -G render,video $LOGNAME
sudo reboot
```

### Install the ubuntu packages 

Needed for the workshop (copied from the [README.md](../README.md))

```
sudo apt update
sudo apt install -y python3 python3-venv python3-pip curl espeak ffmpeg libsndfile1 portaudio19-dev
sudo apt-get install -y libcairo2-dev
sudo apt install -y libgirepository1.0-dev
sudo apt install -y \
    python3-gi \
    gir1.2-gst-plugins-base-1.0 \
    libgstreamer1.0-0 \
    gstreamer1.0-tools \
    gstreamer1.0-plugins-base \
    gstreamer1.0-plugins-good \
    gstreamer1.0-plugins-bad \
    gstreamer1.0-libav
```

### Insall Ollama

```
curl -fsSL https://ollama.com/install.sh | sh
ollama pull qwen3:0.6b
ollama serve
```

### Setup the python environment

```
cd ~
git clone https://github.com/alexhegit/ReachyMiniChat.git
cd ReachyMiniChat
python3 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt

pip install "reachy-mini[mujoco]"
```


## Workshop Steps (for developer)

We have the repo of workshop cloned to ~/ReachyMiniChat and python venv setup in it.

```bash
# Terminal 1 — Start the robot simulator
cd ~/ReachyMiniChat
source venv/bin/activate
export PYGLFW_LIBRARY_VARIANT=x11
reachy-mini-daemon --sim

# Terminal 2 — Activate environment
cd ~/ReachyMiniChat
source venv/bin/activate
```

# verfiy the ollama servic
ollama run qwen3:0.6b "hi"
```

> **Tip:** Every script supports `--help`. Run `python emo_vX.py --help` to see available flags without launching the full pipeline.

---

NOTE: The workshop at Shanghai AI Dev Day is using remote machine throuhg the VNC login.The ASR/TTS and connect to ReachyMini Robot is invalide.
Only the simulation with emo_v1.py, emo_v2.py, emo_v3.py is working within VNC environment.

### v1 — High-Intensity Emotion Engine

**Concept:** Baseline emotion controller with large-amplitude motions triggered by keyword analysis.

```bash
cd ~/ReachyMiniChat

# Quick logic test (no robot required)
python emo_v1.py --test

# Interactive chat (requires daemon + Ollama)
python emo_v1.py --chat
```

**Try:** Type "happy" vs "sad" and observe the motion amplitude.

**Key idea:** Emotions are classified by simple keyword matching; intensity is always "high".

---

### v2 — Recorded Moves Library

**Concept:** Replace custom motions with pre-recorded move libraries from Hugging Face.

```bash
# Play all recorded moves (dances + emotions)
python utils/test_actions.py

# Interactive chat with recorded moves
python emo_v2.py --chat
```

**Try:** Watch the robot perform dances from `pollen-robotics/reachy-mini-dances-library`.

**Key idea:** Moves are categorized by emotion and selected based on LLM output analysis.

---

### v3 — Parallel Actions During Streaming

**Concept:** Trigger actions *while* the LLM is still generating text, not after.

```bash
python emo_v3.py --chat
```

**Try:** Ask a long question. The robot starts moving before the full response is received.

**Key idea:** Actions are triggered on partial response chunks for lower latency.

---

### v4 — Offline TTS (espeak) + Lip-sync

**Concept:** Add deterministic offline speech synthesis using `espeak` with basic lip-sync hooks.

```bash
# Test TTS only
python emo_v4.py --test-tts

# Full chat with speech
python emo_v4.py --chat
```

**Try:** `--no-tts` flag disables speech if you only want motion.

**Key idea:** Audio is generated via `espeak --stdout` and played with `sounddevice`.

---

### v5 — Edge-TTS (Cloud Neural Voices)

**Concept:** Higher-quality multilingual speech via Microsoft Edge-TTS cloud API.

```bash
# Test Edge-TTS voice
python emo_v5.py --test-tts

# Chat with neural voice
python emo_v5.py --chat
```

**Try:** `utils/test_edge_tts_voices.py` discovers cartoon/cute voices across languages.

**Key idea:** WAV is saved at the source sample rate to avoid playback distortion.

---

### v6 — Continuous Synchronized Actions

**Concept:** Actions persist throughout the *entire* speech, with synchronized eye blink + body yaw + head pose + antenna movement.

```bash
# Test synchronized action sequences
python emo_v6.py --test-actions

# Test TTS with cartoon voices
python emo_v6.py --test-tts

# Full experience
python emo_v6.py --chat
```

**Try:** `--gentle` reduces motion amplitude for safe human-robot interaction.

**Key idea:** 4-5 action sequences per emotion; threading coordinates speech and motion.

---

### v7 — ASR → LLM → TTS (Push-to-Talk)

**Concept:** Speak to the robot instead of typing. 4-second fixed recording per utterance.

```bash
# Microphone mode
python emo_v7.py --asr

# Gentler motions for nearby humans
python emo_v7.py --asr --gentle
```

**Try:** Speak clearly; the robot transcribes → queries Ollama → speaks the response.

**Key idea:** `faster-whisper` on CPU transcribes audio; the rest of the pipeline is identical to v6.

---

### v8 — Fully Offline Pipeline (Piper-TTS)

**Concept:** Replace Edge-TTS with Piper-TTS so the entire pipeline works without internet.

```bash
# Text chat with offline voice
python emo_v8.py --chat --piper-model models/en-us-blizzard_lessac-medium.onnx

# ASR + offline voice in English
python emo_v8.py --asr --piper-model models/en-us-blizzard_lessac-medium.onnx --gentle

# ASR + offline voice in Chinese
python emo_v8.py --asr --piper-model models/zh_CN-huayan-medium.onnx --gentle
```

**Try:** Download additional `.onnx` + `.json` voices from [Piper Voices](https://huggingface.co/rhasspy/piper-voices).

**Key idea:** Ollama (local LLM) + Piper-TTS (local TTS) + faster-whisper (local ASR) = fully offline robot.

---

## Quick Command Cheat Sheet

| Goal | Command |
|------|---------|
| Check dependencies | `python emo_v8.py --help` |
| Test robot moves | `python utils/test_actions.py` |
| Test Ollama connection | `python utils/test_ollama_connection.py` |
| Test emotion analysis | `python utils/test_emotion_analysis.py` |
| Measure ASR latency | `python utils/latency_harness.py` |
| Offline chat | `python emo_v8.py --chat --piper-model ...` |

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| `ModuleNotFoundError` | `source .venv/bin/activate && pip install -r requirements.txt` |
| `Connection refused` | Ensure `reachy-mini-daemon --sim` is running |
| No audio / distorted | Install `libsndfile1` and `portaudio19-dev`; verify `sounddevice` in venv |
| Edge-TTS produced empty WAV / "No audio was received." | Retry with default (Chinese) voice for CJK text, or verify network/auth for edge-tts; run `python emo_v5.py --test-tts` to reproduce and inspect logs |
| "Audio system is not initialized." warnings | These are often PortAudio/sounddevice startup warnings; ensure PulseAudio/PipeWire is running or set a device manually: `python -c "import sounddevice as sd; print(sd.query_devices()); sd.default.device = <index>"` |
| Ollama timeout | Run `ollama serve` and verify with `curl http://localhost:11434/api/tags` |
| ASR not working | Check microphone permissions; try `python utils/asr.py` standalone |
| `--help` works but chat fails | Run dependency check: script will print missing packages |

---

## Optional Challenges

1. **Sim-to-Real:** Swap `--sim` for real hardware (`reachy-mini-daemon` without `--sim`).
2. **Custom Voice:** Download a new Piper voice and pass it to `emo_v8.py --piper-model`.
3. **Emotion Tuning:** Modify emotion keywords in `utils/test_emotion_analysis.py` and test with `--interactive`.
4. **Pipeline Timing:** Run `utils/latency_harness.py` to compare fixed vs VAD recording latency.

---

*Workshop duration: ~60-90 minutes for v1→v8. Adjust depth per audience.*
