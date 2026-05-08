# Run OpenClaw with Lemonade Server as the backend

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
- **The final challenge** to your agent.

## Prerequisites

- A PC running **Ubuntu 24.04+** or a compatible Debian-based Linux distribution with `apt-get`
- At least **12 GB of RAM** (32 GB+ recommended for larger models)
- **~10–20 GB of free disk space** for model weights



## Section 1: Install and Start Lemonade Server
### Install Lemonade via the PPA

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

### Verify the Server is Ready

```bash
curl -s http://127.0.0.1:13305/api/v1/models
```

You should see:

```json
{"data":[],"object":"list"}
```

The empty `data` array simply means no model weights have been downloaded yet, the server itself is running and ready.

**Congrats — Lemonade Server is live!** You now have a fully local inference server running on your machine. 

---
### Configuring Context Size

For agent workloads, a larger context window lets the model keep more of the task history, tool outputs, and reasoning steps in view at once. Set this once after the server is running:

```bash
lemonade config set ctx_size=32768
```

This takes effect for newly loaded models. A context of 32768 tokens is a reasonable floor for agent use; increase it if your model and available RAM support it.

---

## Section 2: Install and Configure OpenClaw

### Install OpenClaw

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --no-prompt --no-onboard
```

The `--no-onboard` flag skips the interactive setup wizard, you will configure the model backend manually in the next step, which gives you precise control over which model and server are used.

After installation, confirm `openclaw` is on your `PATH`:

```bash
export PATH="$HOME/.npm-global/bin:$HOME/.local/bin:$PATH"
openclaw --version
```

To persist this across terminal sessions:

```bash
echo 'export PATH="$HOME/.npm-global/bin:$HOME/.local/bin:$PATH"' >> ~/.bashrc
```

### Configure OpenClaw to Use Lemonade

Run OpenClaw's non-interactive onboarding, replacing `YOUR_MODEL_ID` with your Lemonade Model ID. Use the plain name (e.g., `Qwen3.5-9B-GGUF`) for catalog models, or the `user.` prefixed name (e.g., `user.Qwen3.5-9B-GGUF-Q4-K-M`) for custom imported ones:

```bash
openclaw onboard \
  --non-interactive \
  --mode local \
  --auth-choice custom-api-key \
  --custom-base-url "http://127.0.0.1:13305/api/v1" \
  --custom-model-id "YOUR_MODEL_ID" \
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

### Start the OpenClaw Gateway

The gateway is the OpenClaw process that manages the agent loop and serves the dashboard:

```bash
openclaw gateway run --bind loopback --port 18789
```

To open the dashboard, run this in a second terminal while the gateway is still running:

```bash
openclaw dashboard
```

This prints the authenticated URL, open it in your browser. You should see the OpenClaw dashboard with your Lemonade model listed as the active backend. **Your agent is ready.**

<p align="center">
  <img src="https://github.com/user-attachments/assets/fcadf4de-8421-4f14-a63a-2fa5cbb7c4ec" width="500" height="300" />
</p>

**Congratulations — you've built a fully local AI agent stack from scratch.** 

---

---
## Section 3: Fix a bug

### Step 1 - Spot the bug yourself

#### 🖥️ Open a new terminal and run the app

In a new terminal, follow the instructions OpenClaw provides on how to run `open_type_faster`.

Type for 30 seconds. When the results screen appears, look at the accuracy.

> **Something is off. Do you see it?**

### Step 2 - Ask OpenClaw to test

#### 🦞 Ask your OpenClaw agent

Ask your OpenClaw agent to run the tests in the repo to see what's wrong. Here is an example:

```text
"Run the tests in the repo and tell me if something is failing"
```

### Step 3 - Watch the agent fix it

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

### Package as a skill
What the agent just did was a one-off. A **skill** packages that behavior so it can be invoked on any Python project. Skills live in `~/.openclaw/workspace/skills/` as a folder containing a `SKILL.md` file with YAML frontmatter and step-by-step instructions. OpenClaw injects them into the system prompt automatically.

#### 🦞 Ask your OpenClaw agent
Ask your OpenClaw agent to create a python debugger skill. Try sending this to your OpenClaw agent:

```text
"Create a skill called pytest-debugger in the skills directory. The SKILL.md must define these steps: 1) Read the tests/ folder to understand what is being tested. 2) Run pytest with verbose output and short tracebacks. 3) For each failing test, read the source file it references. 4) Identify the minimal fix — do not rewrite functions. 5) Apply the fix and re-run pytest to confirm. 6) Report: file changed, line changed, what was wrong, what the fix was."
```

When done, run the next cell to see what it wrote.

That file is all it takes. The skill lives in the workspace, alongside `IDENTITY.md` and `SOUL.md`, and OpenClaw injects it into every session automatically. Any time you ask the agent to debug a Python project, it has these steps to follow.

Skills are just markdown files. You can read them, edit them, and version-control them.

---
### Using the Skill

The skill is now in the workspace and OpenClaw injects it into the agent's system prompt automatically — no special command needed.

The real value of a skill is applying it to a **completely different codebase**, one the agent has never seen, without writing a single new instruction. There's a broken Python project at `git@github.com:Mahdi-CV/buggy-py-mg.git` — hand it to the agent and watch the skill do the work.

#### 🦞 Ask your OpenClaw agent

```text
Clone git@github.com:Mahdi-CV/buggy-py-mg.git into your workspace, then use the pytest-debugger skill to find and fix whatever is broken.
```

The agent will apply the exact same steps it used on `open_type_faster` — no re-explaining, no extra guidance.


---
## Section 5: Autonomous Workflows

You've built an agent that fixes bugs on demand. Now make it work *for you*: automatically, every morning, without lifting a finger.

As an AI/ML developer, you need to track fast-moving repos (like SGLang, vLLM, Transformers, ROCm, OpenClaw) and keep up with daily AI hardware news. Instead of writing scripts or configuring schedulers or spending hours manually, tell the agent what your ideal morning looks like and let it figure out the rest.

What it can do autonomously:
- Schedule itself to run every morning at 8 AM
- Save your filtering preferences to long-term memory (`MEMORY.md`)
- Check GitHub repos for relevant PRs and changes
- Search the web for AI hardware news
- Write the brief to your workspace

### Step 1 - Delegate (don't mention cron jobs)

#### 🦞 Ask your OpenClaw agent

The prompt below is purely conversational—no mention of cron, no memory commands. The agent can infer what infrastructure it needs from the prompt:

```text
"I need to wake up to a personalized tech brief every morning at 8 AM. Check sgl-project/sglang, vllm-project/vllm, huggingface/transformers, ROCm/ROCm, and openclaw/openclaw. I only care about performance updates, GPU features, and breaking changes — skip CI/infrastructure noise and docs-only PRs. Also search the web and add a summary of the latest AI hardware news."
```

Come back here once the agent confirms it has set up the schedule.

### Step 2 - Peek under the hood

The agent didn't just answer, it acted and wrote things to the disk. Run the next two cells to check it out.

Your filtering rules ("skip CI noise", "only GPU features") are now in `MEMORY.md`. Every future session reads this file at startup. Update your preferences anytime by telling the agent in plain language and it will rewrite its own memory, e.g."actually, include HuggingFace trending models too".

#### 🦞 Ask your OpenClaw agent
Try a run now instead of waiting until 8 AM. Here is an example prompt:

```text
"Run my morning brief right now and show me the output."
```

---
## Section 6: Optional Challenges

### 🦞 Package a different skill with your own daily work purpose.
### 🦞 Explore the various skills in [ClawHub]([https://clawhub.ai/]
### 🦞 Publish your skill in ClawHub 🚀
