---
name: open-claw-net
description: >
  Complete setup guide for a self-sovereign, decentralized AI and services cluster across a
  friend group — no GPUs required, no paid API models, no cloud dependencies. Use this skill
  when someone wants to build a private AI cluster with Raspberry Pis, replace SaaS apps
  (Spotify, Google Drive, Dropbox) with self-hosted alternatives, set up a friend-group mesh
  network, run open-weight AI models with no provider lock-in, or exit reliance on Anthropic,
  OpenAI, or Google for inference. Trigger this for any combination of: "friend group server",
  "self-hosted AI", "Pi cluster", "replace Spotify", "NextCloud setup", "CasaOS", "mesh
  network", "open source LLM no API key", "get off cloud providers", or "decentralized tech
  stack with friends" — even if the user doesn't say all of these things at once.
---

# open-claw-net — Decentralized Friend Group AI + Services Stack

A complete blueprint for a group of technologists who want to stop renting compute and
start owning it. No GPUs required. No paid API keys. No Spotify, Google Drive, or AWS.
Just hardware you own, software you control, and a private mesh network that ties it together.

This is a living stack. You don't build it all at once. You build the foundation, then
layer services on top as the group gets comfortable.

---

## The philosophy in one paragraph

Every SaaS subscription your group pays is rent on infrastructure someone else controls.
Every API key is a dependency on a company whose terms, pricing, and existence you cannot
guarantee. The alternative — owning your stack — used to require a data center budget.
It doesn't anymore. A Raspberry Pi 5 costs $80. Open-weight models run on CPU. Mesh
networking is free. The barrier is knowledge, not money. This guide is the knowledge.

See `inspiration.md` for the communities and projects that proved this was possible.
See `references.md` for the technical reading list.

---

## What you're building

```
Each person's node (Raspberry Pi 5)
    ↓
Tailscale mesh network (private, encrypted, free)
    ↓
Shared inference layer (Petals / llama.cpp distributed)
    ↓
OpenClaw gateway (one per person or one shared)
    ↓
CasaOS home server (one per house or shared VPS)
    ├── NextCloud — files, calendar, contacts
    ├── Navidrome — music (Spotify replacement)
    ├── Matrix/Element — group chat (Discord replacement)
    └── SearXNG — private search
```

---

## Phase 0: How many Raspberry Pis and what kind

### The honest answer on Pi inference

A single Pi 5 (8GB) can run a small model (1–4B parameters) at 2–5 tokens/second.
That's slow compared to a GPU but it's private, it's free after purchase, and it runs
24/7 on 5 watts. For a friend group's async AI tasks — daily briefings, research
summaries, writing help — it's genuinely usable.

The key insight: **don't try to run a big model on one Pi.** Run the right model for
the hardware, or distribute across all Pis using Petals.

### Per-person minimum node

Every person in the group gets their own Pi. This is their personal compute node —
their agent runs here, their data lives here, their node contributes to the collective.

| Component | Model | Cost (approx) |
|---|---|---|
| Single-board computer | Raspberry Pi 5 (8GB) | $80 |
| Storage | 512GB NVMe SSD + M.2 HAT+ | $45 |
| Cooling | Active cooler (official) | $10 |
| Power | 27W USB-C power supply | $15 |
| Case | Any Pi 5 compatible | $10–20 |
| **Total per person** | | **~$160–170** |

For a group of 6: ~$1,000 total. Split that six ways — $167 each, one time, no monthly fee.

### What each Pi can run locally (no network, just your node)

| Model | Size on disk | Tokens/sec on Pi 5 8GB | Best for |
|---|---|---|---|
| Gemma 2 2B (Q8) | ~2.7GB | 4–6 tok/s | Fast everyday tasks |
| Phi-3.5 Mini (Q4) | ~2.4GB | 3–5 tok/s | Reasoning, coding |
| Llama 3.2 3B (Q4) | ~2.0GB | 4–6 tok/s | General assistant |
| Qwen2.5 3B (Q4) | ~2.0GB | 3–5 tok/s | Multilingual, coding |
| Llama 3.2 1B (Q8) | ~1.3GB | 8–12 tok/s | Background tasks, heartbeats |

The small model runs your personal OpenClaw agent. Use the fastest one for background
noise (heartbeats, subagents). Use the smarter one for conversations.

### What the group cluster can run together (Petals distributed inference)

When all Pis contribute to Petals, you get a collective inference pool:

| Group size | Combined RAM | Shared model capability | Speed |
|---|---|---|---|
| 3 people | 3 × 8GB = 24GB | Llama 3.2 8B at full precision | ~1–2 tok/s |
| 6 people | 6 × 8GB = 48GB | Llama 3.1 8B full + Mistral 7B | ~1–2 tok/s |
| 8 people | 8 × 8GB = 64GB | Llama 3.3 70B at Q2 (low quality) | ~0.5 tok/s |
| 10 people | 10 × 8GB = 80GB | Mistral 22B or Qwen2.5 14B Q4 | ~0.8–1 tok/s |

Be honest with your group: collective Pi inference is slow. 1–2 tok/s on an 8B model
means a 500-word response takes ~3 minutes. This is fine for async tasks (morning
briefings, research, batch summarization). It's bad for interactive chat. Use your
local small model for chat. Use the collective pool for heavyweight tasks.

For detailed hardware specs and upgrade paths, see `references/hardware.md`.

---

## Phase 1: Software prerequisites per person

Every person installs this on their Pi before the group does anything together.

### Operating system

```bash
# Flash Raspberry Pi OS Lite (64-bit, no desktop) to your NVMe
# Boot from NVMe, not SD card — much faster
# Enable SSH, set hostname to something memorable (alice-node, bob-node, etc.)

# Update everything first
sudo apt update && sudo apt full-upgrade -y
sudo reboot
```

### Core dependencies

```bash
# Build tools for llama.cpp and Python packages
sudo apt install -y git curl wget build-essential cmake python3 python3-pip \
  python3-venv libopenblas-dev screen tmux htop

# Node.js (for OpenClaw)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Docker (for CasaOS services)
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
newgrp docker
```

### Ollama (primary inference runtime)

```bash
curl -fsSL https://ollama.com/install.sh | sh

# Pull your two models: one smart, one fast
ollama pull phi3.5:3.8b-mini-instruct-q4_K_M    # your personal assistant
ollama pull llama3.2:1b-instruct-q8_0            # background tasks

# Verify
ollama list
ollama run phi3.5:3.8b-mini-instruct-q4_K_M "hello, are you working?"
```

### llama.cpp (for Petals and fine-grained control)

```bash
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp
cmake -B build -DLLAMA_BLAS=ON -DLLAMA_BLAS_VENDOR=OpenBLAS
cmake --build build --config Release -j4
# Building takes ~20 minutes on Pi 5 — start it, go make coffee
```

### OpenClaw

```bash
npm install -g openclaw
openclaw setup
# When prompted for model: point at local Ollama
# Model string: ollama:phi3.5:3.8b-mini-instruct-q4_K_M
```

### Petals (for collective inference)

```bash
python3 -m venv ~/petals-env
source ~/petals-env/bin/activate
pip install petals
```

For detailed Petals cluster setup and which models to serve, see `references/inference-stack.md`.

---

## Phase 2: Mesh network (Tailscale)

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up --advertise-routes=192.168.x.0/24  # advertise your local LAN
```

One person creates the Tailscale account (free, up to 100 devices). Share the invite.
Everyone joins. Each Pi gets a stable Tailscale hostname like `alice-node.tail12345.ts.net`.

Verify: every Pi should be able to `ping <other-node>.tail12345.ts.net` successfully.

**Pushing configs to all nodes at once:**
```bash
# Install Ansible on your machine (not the Pis)
pip install ansible

# Create inventory file
cat > inventory.ini << EOF
[nodes]
alice-node.tail12345.ts.net
bob-node.tail12345.ts.net
charlie-node.tail12345.ts.net
EOF

# Push a config change to all nodes simultaneously
ansible all -i inventory.ini -m copy \
  -a "src=openclaw-config.json dest=~/.openclaw/config.json"

# Or run a command on all nodes
ansible all -i inventory.ini -m shell -a "openclaw restart"
```

For mesh architecture options beyond Tailscale (Yggdrasil, cjdns), see `references/mesh-network.md`.

---

## Phase 3: OpenClaw — no paid models

This is the critical difference from guides that assume API keys. Every model reference
points at your local Ollama instance. No Anthropic key. No OpenAI key. No bill.

### Gateway config (on each person's Pi, or one shared node)

```json
{
  "gateway": {
    "host": "100.x.x.x",
    "port": 18789
  },
  "ai": {
    "baseURL": "http://127.0.0.1:11434/v1",
    "model": "ollama:phi3.5:3.8b-mini-instruct-q4_K_M",
    "modelOverrides": {
      "heartbeat": "ollama:llama3.2:1b-instruct-q8_0",
      "subagent": "ollama:llama3.2:1b-instruct-q8_0",
      "heavy": "petals:meta-llama/Llama-3.1-8B-Instruct"
    }
  }
}
```

The `heavy` override routes compute-intensive tasks to the Petals collective pool.
Your Pi's small model handles everything interactive. The pool handles the hard stuff.

### Lock it down before anything else

```bash
openclaw doctor   # fix all warnings before connecting any channels
```

Gateway bound to Tailscale IP only. Never 0.0.0.0 on a network-connected machine.

---

## Phase 4: Shared services stack (CasaOS + NextCloud)

One person (or a shared cheap VPS, ~$6/month) runs CasaOS as the group's app server.
Everyone else connects via Tailscale.

### Install CasaOS

```bash
curl -fsSL https://get.casaos.io | sudo bash
# Access the UI at http://<tailscale-ip>:80 after install
```

CasaOS gives you a Docker app store via a browser UI. Every service below installs in
2 clicks from the CasaOS dashboard.

### NextCloud — replace Google Drive / Dropbox / iCloud

Install from CasaOS app store, or manually:

```yaml
# docker-compose.yml
services:
  nextcloud:
    image: nextcloud:latest
    ports:
      - "8080:80"
    volumes:
      - nextcloud_data:/var/www/html
    environment:
      - NEXTCLOUD_ADMIN_USER=admin
      - NEXTCLOUD_ADMIN_PASSWORD=changeme
      - NEXTCLOUD_TRUSTED_DOMAINS=your-node.tail12345.ts.net
```

NextCloud gives your group: shared file storage, calendar sync, contacts sync, photo
backup, collaborative documents, and video calls. Everything Google Workspace does.

### Navidrome — replace Spotify

Navidrome is a self-hosted music server. Add your group's collective music library
(FLAC, MP3, whatever you have), and every member streams it from any device via
any Subsonic-compatible app.

```yaml
services:
  navidrome:
    image: deluan/navidrome:latest
    ports:
      - "4533:4533"
    volumes:
      - /path/to/music:/music:ro
      - navidrome_data:/data
    environment:
      - ND_SCANSCHEDULE=1h
      - ND_LOGLEVEL=info
```

Client apps: Symfonium (Android), Amperfy (iOS), Feishin (desktop). All free.
Each person syncs their local music library to the shared Navidrome server. The
group collectively builds a music library that everyone can access. No algorithm.
No subscription. No ads.

### Matrix / Element — replace Discord / WhatsApp

```bash
# Synapse Matrix homeserver
docker run -d \
  --name synapse \
  -p 8448:8448 \
  -v ~/synapse:/data \
  matrixdotorg/synapse:latest
```

Your group gets end-to-end encrypted chat, voice calls, and file sharing on a server
you control. Anyone can join your Matrix server from any Matrix client (Element, Cinny,
Schildichat). Bridges available for Signal, Telegram, WhatsApp if needed.

### SearXNG — replace Google Search

```bash
docker run -d \
  --name searxng \
  -p 8888:8080 \
  searxng/searxng:latest
```

Point OpenClaw's web search skill at `http://<tailscale-ip>:8888`. Your group's search
queries now go to a meta-search engine on your own hardware.

For full CasaOS app configurations and the complete self-hosted services catalogue,
see `references/self-hosted-services.md`.

---

## Phase 5: Getting fully off model providers

This is the deeper question: how does a group of technologists achieve complete
independence from Anthropic, OpenAI, Google, and Meta for AI inference?

### The short answer

Open-weight models are already good enough for most tasks. Llama 3.1 8B running on
your Pis handles 80% of what people use Claude or GPT for. The remaining 20% — complex
reasoning, long-context analysis — gets better every 3–6 months as open-weight models
improve. The exit ramp exists now. You just have to take it.

### The complete open stack

| Layer | What you need | What to use |
|---|---|---|
| Model weights | Open-weight, downloadable | Llama 3.x, Mistral, Phi-3, Qwen2.5, Gemma 2 |
| Inference runtime | Runs models on your hardware | Ollama, llama.cpp, Petals |
| Agent platform | OpenClaw, pointed at local inference | Already covered above |
| Search | Not Google | SearXNG (self-hosted) |
| Memory/storage | Not Google Drive | NextCloud |
| Communication | Not Discord/WhatsApp | Matrix/Element |
| Code hosting | Not GitHub (owned by Microsoft) | Gitea (self-hosted) |
| DNS | Not Google 8.8.8.8 | Pi-hole + Unbound |

### Which open models to trust and why

**Llama 3.x (Meta):** MIT-licensed for models under 70B, custom license for larger.
Excellent quality. The 8B model is the workhorse — genuinely capable, runs on 8GB RAM.

**Mistral / Mixtral (Mistral AI):** Apache 2.0 license, truly open. French company,
released under genuine open-source terms. 7B model punches above its weight.

**Phi-3.5 Mini (Microsoft):** MIT licensed. Surprisingly capable for its 3.8B size.
Best single-Pi model for reasoning tasks.

**Qwen2.5 (Alibaba):** Apache 2.0. Excellent for coding and multilingual. Strong
performance in the 3B–14B range.

**Gemma 2 (Google):** Gemma license (permissive for research/commercial under limits).
Very efficient — 2B model beats many 7B models on reasoning benchmarks.

**Who to avoid:** Any model where you can't download the weights and run them
offline. "Open" models that require an API call aren't open.

### Fine-tuning your own model on group data

Once you have base infrastructure working, the advanced move is fine-tuning a small
model on your group's collective knowledge base — your notes, documents, workflows,
shared understanding. This creates a model that actually knows your context.

```bash
# Use Unsloth for Pi-friendly fine-tuning (low VRAM/RAM optimized)
pip install unsloth

# Fine-tune Llama 3.2 3B on your group's markdown files
# Takes ~2-6 hours on Pi cluster via Petals
# See references/open-models.md for the full walkthrough
```

For the complete open model selection guide and fine-tuning walkthrough,
see `references/open-models.md`.

For the full provider exit strategy and timeline, see `references/exit-strategy.md`.

---

## Phase 6: SOUL.md for each person

Same as any OpenClaw setup — but with one addition for a local-model setup: set
realistic expectations about what your small model can do.

```markdown
you are [name]. you assist [person].

be direct. match my tone. answer first, elaborate only if asked.
never say "absolutely", "certainly", or "great question."
if you don't know something, say so.

you are running on a small local model (phi-3.5 mini or similar).
you don't have internet access by default.
for complex research tasks, say you'll hand off to the group's shared model.
for tasks needing web search, use the SearXNG tool.

never create accounts without my explicit approval.
never delete files or messages without asking first.
never share my data with external services.
```

---

## Quick start checklist

- [ ] Every person has Pi 5 (8GB) with NVMe SSD, imaged and booted
- [ ] All core dependencies installed (Node.js, Docker, Python 3, cmake)
- [ ] Ollama installed, two models pulled (smart + fast)
- [ ] llama.cpp compiled (takes 20 min — do it first, walk away)
- [ ] Everyone on Tailscale, ping tests pass between all nodes
- [ ] OpenClaw installed, pointed at local Ollama, gateway bound to Tailscale IP
- [ ] `openclaw doctor` shows no critical warnings
- [ ] Model overrides configured — heartbeat/subagent on fast 1B model
- [ ] Petals running on all nodes (even background)
- [ ] CasaOS running on host/VPS node
- [ ] NextCloud accessible from all nodes via Tailscale
- [ ] Navidrome running, music library synced
- [ ] Matrix homeserver running, everyone has accounts
- [ ] SearXNG running, OpenClaw web search pointed at it
- [ ] Ansible playbook working (can push config to all nodes at once)
- [ ] Everyone has written their SOUL.md
- [ ] Skills locked to verified sources only

---

## Reference files

- `references/hardware.md` — Pi 5 specs, upgrade paths, what to buy
- `references/inference-stack.md` — Ollama, llama.cpp, Petals setup in detail
- `references/open-models.md` — Model selection, licenses, fine-tuning your own
- `references/self-hosted-services.md` — Full CasaOS stack, NextCloud, Navidrome, Matrix
- `references/mesh-network.md` — Tailscale, Yggdrasil, Ansible config push
- `references/exit-strategy.md` — Full provider independence roadmap and timeline
- `references/security.md` — Multi-user security, per-user permissions
- `references/migration.md` — Clawdbot/Moltbot migration for early adopters
- `inspiration.md` — Communities, projects, and people who proved this was possible
- `references.md` — Technical reading list, papers, documentation
