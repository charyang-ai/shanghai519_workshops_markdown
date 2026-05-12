# Run OpenClaw with Lemonade Local LLM Serving in AMD Ryzen AI Device

## Overview

[**OpenClaw**](https://openclaw.ai/) is an autonomous AI agent that can write and run code, manage files, and work through complex multi-step tasks on your behalf. Unlike a chat assistant that just answers questions, OpenClaw takes real actions on your system, which means it needs a fast, capable AI backend that can keep up with a demanding agent loop.

[**Lemonade Server**](https://lemonade-server.ai/) is that backend. It is an open-source local inference server that runs GenAI models directly on your hardware and exposes them through the industry-standard OpenAI API.

Together, they form a fully local AI agent stack: Lemonade handles model inference, and OpenClaw provides the agent loop that turns model outputs into real actions.

> **Before you continue:** OpenClaw is a highly autonomous AI agent. Giving any AI agent access to your system may result in unpredictable or unintended outcomes. Proceed only if you understand the risks and are comfortable with autonomous software acting on your behalf.

## What You'll Learn Today

By the end of this playbook you will be able to:

- Learn about **Lemonade Server** and raise a API service.
- **Install OpenClaw** and **point it at Lemonade Server** as its AI backend.
- **Fix a bug** to let your agent learn to fix bug and improve by itself.
- **Package a skill** to your agent so you can use it in general sessions.
- **Autonomous Workflows** make it work for you: automatically, every morning, without lifting a finger.
- **Optional challenges** to your agent.

## Prerequisites

- A PC running **Ubuntu 24.04+** or a compatible Debian-based Linux distribution with `apt-get`
- At least **32 GB of RAM** (96 GB recommended for larger models)
- **50+ GB of free disk space** for model weights



## Section 1: Install and Start Lemonade Server
### Step 1 - Install Lemonade via the PPA

```bash
sudo apt-get install -y software-properties-common
sudo add-apt-repository -y ppa:lemonade-team/bleeding-edge
sudo apt-get update
sudo apt-get install -y lemonade-server
```

This installs two key binaries:

| Binary | Role |
|--------|------|
| `lemonade` | CLI for model management: list, pull, import, configure |
| `lemond` | Daemon that hosts the HTTP inference server |

Confirm the install worked:

```bash
lemonade --version
```
<img width="640" height="61" alt="image" src="https://github.com/user-attachments/assets/c6b9d984-9d35-42cf-a1e6-124545004993" />

### Step 2 - Check the available LLM models list 
(Qwen3.6 was pre-downloaded, we will use it as our llm backend)

```bash
lemonade list
```
<img width="640" height="280" alt="image" src="https://github.com/user-attachments/assets/052e71b3-d14f-4427-99da-004dadaf3d5a" />


### Step 3 - Serve the Qwen3.6-35B model with local GPU

```bash
lemonade run Qwen3.6-35B-A3B-GGUF
```
<img width="640" height="145" alt="image" src="https://github.com/user-attachments/assets/63a2f49b-3acb-4c1b-97b0-18dcb2c9a804" />

### Step 4 - Verify the Server is Ready

```bash
sudo apt install curl
curl -s http://127.0.0.1:13305/api/v1/models
```

You should see:

```json
{"data":[],"object":"list"}
```

The empty `data` array simply means no model weights have been downloaded yet, the server itself is running and ready.

**Congrats — Lemonade Server is live!** You now have a fully local inference server running on your machine. 

---
### Step 5 - Configuring Context Size

For agent workloads, a larger context window lets the model keep more of the task history, tool outputs, and reasoning steps in view at once. Set this once after the server is running:

```bash
lemonade config set ctx_size=190000
```

This takes effect for newly loaded models. A context of 32768 tokens is a reasonable floor for agent use; increase it if your model and available RAM support it.

---

## Section 2: Install and Configure OpenClaw

### Step 1 - Install OpenClaw

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --no-prompt --no-onboard
```

The `--no-onboard` flag skips the interactive setup wizard, you will configure the model backend manually in the next step, which gives you precise control over which model and server are used.
<img width="640" height="100" alt="image" src="https://github.com/user-attachments/assets/0cf32d92-7c0b-41ac-84e4-aac7511f8ab3" />

After installation, confirm `openclaw` is on your `PATH`:

```bash
export PATH="$HOME/.npm-global/bin:$HOME/.local/bin:$PATH"
openclaw --version
```
<img width="640" height="85" alt="image" src="https://github.com/user-attachments/assets/4b91692f-12eb-450d-b2da-91be05fa31be" />

To persist this across terminal sessions:

```bash
echo 'export PATH="$HOME/.npm-global/bin:$HOME/.local/bin:$PATH"' >> ~/.bashrc
```

### Step 2 - Configure OpenClaw to Use Lemonade

Run OpenClaw's non-interactive onboarding. If you want to use some other models, replacing 'Qwen3.6-35B-A3B-GGUF' with your Lemonade Model ID. Use the plain name (e.g., `Qwen3.6-35B-A3B-GGUF`) for catalog models, or the `user.` prefixed name (e.g., `user.Qwen3.6-35B-A3B-GGUF-Q4-K-M`) for custom imported ones:

```bash
openclaw onboard \
  --non-interactive \
  --mode local \
  --auth-choice custom-api-key \
  --custom-base-url "http://127.0.0.1:13305/api/v1" \
  --custom-model-id Qwen3.6-35B-A3B-GGUF \
  --custom-provider-id "lemonade" \
  --custom-compatibility "openai" \
  --custom-api-key "lemonade" \
  --secret-input-mode plaintext \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --skip-health \
  --accept-risk
```

This command writes OpenClaw's configuration to `~/.openclaw/openclaw.json`.
<img width="640" height="140" alt="image" src="https://github.com/user-attachments/assets/3ce8ca93-5671-49ec-bf03-e5a085a5c2b3" />

### Step 3 - Start the OpenClaw Gateway

The gateway is the OpenClaw process that manages the agent loop and serves the dashboard:

```bash
openclaw gateway run --bind loopback --port 18789
```

To open the dashboard, run this in a second terminal while the gateway is still running:

```bash
openclaw dashboard
```

This prints the authenticated URL, open it in your browser. You should see the OpenClaw dashboard with your Lemonade model listed as the active backend. **Your agent is ready.**
#### Turn off the brain icon '🧠' to close model thinking. 
<p align="center">
  <img src="https://github.com/user-attachments/assets/fcadf4de-8421-4f14-a63a-2fa5cbb7c4ec" width="500" height="300" />
</p>

**Congratulations — you've built a fully local AI agent stack from scratch.** 

The OpenClaw interface will appear and once it says `gateway connected`, it's ready!
This is where you will chat with your OpenClaw agent whenever you see the 🦞 emoji.

### Step 4 - Configure your OpenClaw
SETTINGS -> AI & Agents -> Models -> Model Provider Context Tokens & Window (set to 65535)
<img width="400" height="400" alt="image" src="https://github.com/user-attachments/assets/cb3721f8-c74a-4798-8eb1-d23bc0ab0dec" />
<img width="400" height="400" alt="image" src="https://github.com/user-attachments/assets/a8a1afdb-cbcd-48f3-824b-45f7762133b8" />


### Step 5 - Start to co-work with your OpenClaw

#### 🦞 Ask your OpenClaw agent

Send your first message to your OpenClaw agent and get setup! Answer any setup questions your agent asks you and then come back here to learn what happened behind the scenes.

#### 👀 Behind the scenes — what just happened?

Those answers didn't just disappear. The agent's memory, personality, and skills are written in markdown files. You can read them, edit them, and version-control them. The agent wrote them to files in its **workspace**, a folder it reads on every message to remember who you are and how to behave.

The workspace has 8 files that get injected into every session:

```
~/.openclaw/workspace/
├── SOUL.md       ← WHO the agent is: values, personality, tone
├── AGENTS.md     ← HOW it operates: startup rules, memory, red lines
├── IDENTITY.md   ← WHAT others see: name, emoji, public metadata
├── USER.md       ← WHO you are: context the agent reads about you
├── TOOLS.md      ← local environment notes (SSH hosts, device names, etc.)
├── MEMORY.md     ← long-term curated memory across sessions
├── HEARTBEAT.md  ← checklist for periodic background checks
├── BOOTSTRAP.md  ← first-run ritual (deleted after onboarding)
└── memory/       ← daily session logs (today + yesterday auto-loaded)
```
### Try it: manually layer a rule into AGENTS.md

`SOUL.md` is personality, don't touch that. `AGENTS.md` is policy. You can manually add rules to the agent here as well. For example, about *how* to behave when debugging.

The next cell appends a debugging policy to `AGENTS.md`. After this, every debug explanation from the agent will follow these rules. Run it to see the before and after.

**SOUL.md is personality. AGENTS.md is policy.** Every message the agent receives, it re-reads both. Run the next cell to see all three key files at once.

Ask OpenClaw add below 'debug rule' policy to AGENTS.md 

```text
Append below 'debug rule' policy to AGENTS.md

## Debugging Policy (added during workshop)

When investigating and explaining bugs:
- State the bug in two sentences max: file, line, what it does wrong
- No filler phrases — just the root cause and the fix
- Use bullet points, never paragraphs
- After fixing, report exactly: file changed, line changed, before, after
```
This is the pattern for real use: you don't replace the soul for every task, you layer specialized instructions on top of a stable base.

> **Note:** You will see this debugging policy in action in Section 3.

## Section 3: Fix a bug

---
### Step 1 - Pull the App

Now let's give the agent something to work with. Ask the agent to clone the typing app and set it up.

#### 🦞 Ask your OpenClaw agent
Ask your OpenClaw agent to clone the `charyang-ai/open_type_faster` repo and explain it to you. Here is an example:

```text
"Clone https://github.com/charyang-ai/open_type_faster, install its dependencies, and tell me how to run it"
```

Watch as your agent uses shell tools to clone, inspect requirements, and install libraries. This is the **act** phase — the agent doesn't just answer, it does.


### Step 2 - Spot the bug yourself

#### 🖥️ Open a new terminal and run the app

In a new terminal, follow the instructions OpenClaw provides on how to run `open_type_faster`.

Type for 30 seconds. When the results screen appears, look at the accuracy.

> **Something is off. Do you see it?**

### Step 3 - Ask OpenClaw to test

#### 🦞 Ask your OpenClaw agent

Ask your OpenClaw agent to run the tests in the repo to see what's wrong. Here is an example:

```text
"Run the tests in the repo and tell me if something is failing"
```

### Step 4 - Watch the agent fix it

#### 🦞 Ask your OpenClaw agent

Ask your OpenClaw agent to fix the failures and run the tests again. Here is an example:

```text
"Fix the failures and rerun the tests"
```

Watch the loop:

1. **Run** `pytest`: see which tests fail and what they expect
2. **Read** the failing test: understand the correct behavior
3. **Read** `stats.py`: trace the calculation back to the bug
4. **Fix** the minimal change needed
5. **Re-run** to verify

This think → act → observe → repeat loop is what separates an agent from a chatbot.

---
## Section 4: Package as a Skill

What the agent just did was a one-off. A **skill** packages that behavior so it can be invoked on any Python project. Skills live in `~/.openclaw/workspace/skills/` as a folder containing a `SKILL.md` file with YAML frontmatter and step-by-step instructions. OpenClaw injects them into the system prompt automatically.

#### 🦞 Ask your OpenClaw agent
Ask your OpenClaw agent to create a python debugger skill. Try sending this to your OpenClaw agent:

```text
"Create a skill called pytest-debugger in the skills directory. The SKILL.md must define these steps: 1) Read the tests/ folder to understand what is being tested. 2) Run pytest with verbose output and short tracebacks. 3) For each failing test, read the source file it references. 4) Identify the minimal fix — do not rewrite functions. 5) Apply the fix and re-run pytest to confirm. 6) Report: file changed, line changed, what was wrong, what the fix was."
```

The agent should show you where the skills file is saved. You can always ask it again to show you.

That file is all it takes. The skill lives in the workspace, alongside `IDENTITY.md` and `SOUL.md`, and OpenClaw injects it into every session automatically. Any time you ask the agent to debug a Python project, it has these steps to follow.

* Skills are just markdown files. You can read them, edit them, and version-control them.

---
## Section 5: Using the Skill

The skill is now in the workspace and OpenClaw injects it into the agent's system prompt automatically — no special command needed.

The real value of a skill is applying it to a **completely different codebase**, one the agent has never seen, without writing a single new instruction. There's a broken Python project at `git@github.com:Mahdi-CV/buggy-py-mg.git`, hand it to the agent and watch the skill do the work.

#### 🦞 Ask your OpenClaw agent

```text
"/skill pytest-debugger git@github.com:Mahdi-CV/buggy-py-mg.git"
```

* The agent will apply the exact same steps it used on `open_type_faster', no re-explaining, no extra guidance.

---
## Section 6: Adding Another Agent for Autonomous Workflows 

You've built an agent that fixes bugs on demand. Now make it work *for you*: automatically, every morning, without lifting a finger.

As an AI/ML developer, you need to track fast-moving repos (like SGLang, vLLM, Transformers, ROCm, OpenClaw) and keep up with daily AI hardware news. Instead of writing scripts or configuring schedulers or spending hours manually, tell the agent what your ideal morning looks like and let it figure out the rest.

What it can do autonomously:
- Schedule itself to run every morning at 8 AM
- Save your filtering preferences to long-term memory (`MEMORY.md`)
- Check GitHub repos for relevant PRs and changes
- Search the web for AI hardware news
- Write the brief to your workspace

#### 🖥️ Open a new terminal and run the app

```bash
openclaw agents add morning-brief
```
OpenClaw will walk you through setup. **Choose these settings:**

1. **Workspace directory** → `/root/.openclaw/workspace/morning-brief`
2. **Copy auth profiles from "main"?** → `Yes`
3. **Configure model/auth for this agent now?** → `No`
4. **Configure chat channels now?** → `No`

### Step 1 - Switch to your new agent

#### 🦞 Ask your OpenClaw agent
```text
"/agent morning-brief"
```

### Step 2 - Delegate 

The prompt below is purely conversational—no mention of cron, no memory commands. The agent can infer what infrastructure it needs from the prompt:

#### 🦞 Ask your OpenClaw agent
```text
"I need to wake up to a personalized tech brief every morning at 8 AM. Check sgl-project/sglang, vllm-project/vllm, huggingface/transformers, ROCm/ROCm, and openclaw/openclaw. I only care about performance updates, GPU features, and breaking changes, skip CI/infrastructure noise and docs-only PRs. Also search the web and add a summary of the latest AI hardware news."
```

---
## Section 6: Optional Challenges

### 🦞 Package a different skill with your own daily work purpose.
### 🦞 Explore the various skills in [ClawHub]([https://clawhub.ai/]
### 🦞 Publish your skill in ClawHub 🚀
