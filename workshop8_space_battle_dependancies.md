# Space Battle — Agentic Game Dev Workshop Dependancies

---

## Prerequisites

### Pre-install Lemonade Server
Lemonade runs large AI models locally on your AMD GPU via ROCm.
Full details at: **https://lemonade-server.ai**

```bash
sudo add-apt-repository ppa:lemonade-team/stable
sudo apt install lemonade-server
```

### Pre-install OpenCode
```bash
curl -fsSL https://opencode.ai/install | bash
```

### Pre-install a modern browser (Google Chrome)
```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo apt install ./google-chrome-stable_current_amd64.deb
```

### Pre-clone the repository
```bash
git clone https://github.com/iswaryaalex/space-battle-agentic-game-dev.git
cd space-battle-agentic-game-dev
```
The repository is pre-cloned at:

```
~/space-battle-agentic-game-dev/
```

### Pre-download model
Please pre-download 2 models here, one is `Qwen3.5-35B-A3B-NoThinking`, the other is `Qwen3.6-35B-A3B-NoThinking`.

From inside the project folder:

```bash
cd ~/space-battle-agentic-game-dev/
lemonade launch opencode
```

Lemonade starts the inference server and launches OpenCode in one command.
You will be prompted to select a model recipe:

```
Select a recipe to import and use with Opencode:
  0) Browse downloaded models
  1) GLM-4.7-Flash-GGUF-NoThinking.json
  2) GLM-4.7-Flash-GGUF-ThinkingCoder.json
  3) Gemma-4-26B-A4B-NoThinking.json
  4) Gemma-4-26B-A4B-ThinkingCoder.json
  5) Qwen3.5-35B-A3B-NoThinking.json
  6) Qwen3.5-35B-A3B-ThinkingCoder.json
  7) Qwen3.6-35B-A3B-NoThinking.json
  8) Qwen3.6-35B-A3B-ThinkingCoder.json
```

**Select option 6 — `Qwen3.5-35B-A3B-ThinkingCoder`. This will help to download model in background**

After option 6 — `Qwen3.5-35B-A3B-ThinkingCoder` finished download, then Ctrl-C to exit and relaunch:

```bash
lemonade launch opencode
```

```
Select a recipe to import and use with Opencode:
  0) Browse downloaded models
  1) GLM-4.7-Flash-GGUF-NoThinking.json
  2) GLM-4.7-Flash-GGUF-ThinkingCoder.json
  3) Gemma-4-26B-A4B-NoThinking.json
  4) Gemma-4-26B-A4B-ThinkingCoder.json
  5) Qwen3.5-35B-A3B-NoThinking.json
  6) Qwen3.5-35B-A3B-ThinkingCoder.json
  7) Qwen3.6-35B-A3B-NoThinking.json
  8) Qwen3.6-35B-A3B-ThinkingCoder.json
```

**Select option 8 — `Qwen3.6-35B-A3B-ThinkingCoder`. This will help to download model in background**

## Result / Validation

### Confirm agents are loaded

Inside OpenCode, type:

```
/agents
```

You should see:
```
ui-renderer
physics-movement
gameplay-rules
vfx-polish      (optional)
boss-ai         (optional)
```

You are all set. Then you can start to build the game.
