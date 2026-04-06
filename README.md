# 🍳 Kitchen Table Cloud

> **Bring tech back to the kitchen table.**

Community cooperative infrastructure — AI, files, music, messaging, and more — owned, governed, and maintained by the people who use it. No venture capital. No algorithm. No rent-seeking.

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Community: Open](https://img.shields.io/badge/community-open-blue.svg)](#get-involved)
[![Hardware: Raspberry Pi 5](https://img.shields.io/badge/hardware-Raspberry%20Pi%205-red.svg)](docs/hardware.md)

---

## In Three Commands

```bash
# Install the AI runtime
curl -fsSL https://ollama.com/install.sh | sh

# Pull a capable free model (2.4GB — less than one month of ChatGPT Plus)
ollama pull phi3.5:3.8b-mini-instruct-q4_K_M

# Talk to it. On your hardware. Privately. Forever.
ollama run phi3.5:3.8b-mini-instruct-q4_K_M "Hello. Are you mine now?"
# It says: Yes. Completely.
```

Cancel a subscription. Come back for the rest.

---

## What This Is

Every major tech platform — Spotify, Google Drive, ChatGPT, Discord — is infrastructure you depend on but don't control. The prices go up. The terms change. Your data trains their models. You have no vote.

Kitchen Table Cloud is a blueprint and community for building the alternative: community-owned, cooperatively governed digital infrastructure that's as capable as the commercial stack but ruled by its users, not shareholders.

The full technical guide — 3,000+ lines, seven parts, full appendices — is in **[sovereign-stack-guide.md](https://meldfunction.github.io/kitchen-table/sovereign-stack-guide.md)**. Check out the **[read me](https://meldfunction.github.io/kitchen-table/sovereign-stack-README.md)** as well.

The human-readable version is at **[Kitchen Table](https://meldfunction.github.io/kitchen-table/)**.

---

## What You Can Run

| Category | Corporate Version | Community Version | Difficulty |
|---|---|---|---|
| AI Assistant | ChatGPT, Claude | Ollama + open models | Easy |
| Distributed AI | GPT-4 API cluster | Petals + 6× Pi 5 nodes | Medium |
| File Storage | Google Drive | NextCloud | Medium |
| Music | Spotify | Navidrome | Easy |
| Photos | Google Photos | Immich | Medium |
| Video Library | Netflix, Plex | Jellyfin | Easy |
| Chat | Discord, WhatsApp | Matrix + Element | Medium |
| Smart Home | Nest, Alexa | Home Assistant | Medium |
| Search | Google | SearXNG | Easy (30 min) |
| Passwords | 1Password, LastPass | Vaultwarden | Easy |
| DNS + Ad Blocking | Google 8.8.8.8 | Pi-hole + Unbound | Medium |
| Off-Grid Comms | Cell signal | Meshtastic LoRa | Medium |
| Code Hosting | GitHub (Microsoft) | Gitea | Medium |
| Home Dashboard | — | CasaOS | Easy |

---

## Repository Structure

```
kitchen-table-cloud/
├── sovereign-stack-guide.md  ← THE GUIDE — full technical walkthrough (Parts 1–7 + Appendix)
├── stitch_guide.py           ← Python build script that assembles the guide from source files
├── docs/
│   ├── hardware.md           ← Pi 5 specs, NVMe boot, cluster builds
│   ├── inference-stack.md    ← Ollama, llama.cpp, Petals setup in detail
│   ├── open-models.md        ← Model selection, licenses, fine-tuning
│   ├── self-hosted-services.md ← Full service configs: NextCloud, Matrix, Navidrome...
│   ├── mesh-network.md       ← Tailscale ACLs, Ansible, Yggdrasil
│   └── exit-strategy.md      ← Phase-by-phase provider independence roadmap
├── config/
│   ├── openclaw-base-config.json
│   ├── tailscale-acls.json
│   └── ollama-config.json
├── playbooks/
│   ├── onboard-new-member.yml  ← Ansible: full Pi setup from scratch
│   ├── update-all.yml          ← Push updates to all nodes simultaneously
│   └── deploy-services.yml     ← Deploy full service stack
├── source-skills/
│   ├── open-claw/              ← OpenClaw configuration skill
│   └── open-claw-net/          ← Distributed Pi cluster skill
└── index.html                  ← GitHub Pages main site
```

---

## Guide Structure

The [sovereign-stack-guide.md](https://meldfunction.github.io/kitchen-table/sovereign-stack-guide.md) covers everything:

| Part | Topic | What You Get |
|---|---|---|
| 1 | OpenClaw basics | CLAUDE.md, permissions, sessions, subagents, hooks, MCP |
| 2 | Advanced configuration | Deep CLAUDE.md patterns, hook recipes |
| 3 | Sovereign stack | Pi hardware, OS, Ollama, llama.cpp, Petals, gateway config |
| 4 | Self-hosted services | NextCloud, Navidrome, Matrix, SearXNG, Gitea full configs |
| 5 | Open models | Which to use, licenses, fine-tuning on group data |
| 6 | Mesh networking | Tailscale ACLs, Ansible, Yggdrasil deep dive |
| 7 | Inspiration + references | Communities, projects, full reading list |
| Appendix | Ready-to-run | Quick-start checklist, cost breakdown, security checklist, migration |

---

## Hardware

**Per person:** Raspberry Pi 5 (8GB) + 512GB NVMe SSD + M.2 HAT+ + active cooler + 27W PSU ≈ **~$160 all-in**

**What a group can do together:**

| Group Size | Collective RAM | AI Capability |
|---|---|---|
| 1 person | 8GB | Phi-3.5 Mini, Llama 3.2 3B — handles 80% of everyday tasks |
| 3 people | 24GB | Llama 3.1 8B at full precision via Petals |
| 6 people | 48GB | Llama 3.1 8B + Mistral 7B simultaneously |
| 10 people | 80GB | Qwen2.5 14B Q4 — approaching frontier capability |

**Ongoing cost:** ~$4/month electricity per node vs $67–109/month in subscriptions replaced.

---

## Security

Read [security.html](https://meldfunction.github.io/kitchen-table/security.html) before opening access to your community.

Key points:
- **Gateway must bind to Tailscale IP only** — never 0.0.0.0 (CVE-2026-25253)
- **Skills locked to verified sources** — ClawHavoc campaign compromised 1,400+ unverified skills
- **Run `openclaw doctor`** before connecting any channels
- **SOUL.md per user** — explicit "never do this" boundaries for every member's agent

---

## Get Involved

This is a **cooperative**, not a startup. Five ways to participate:

1. **Supporter** — Free. Tell people this exists. Share it. Show up.
2. **Member** — Pay-what-you-can. Access all community-run services.
3. **Node Runner** — Contribute hardware. Your Pi = community infrastructure. Electricity stipend.
4. **Builder** — Contribute skills: tech, writing, teaching, design. Your expertise is infrastructure.
5. **Steward** — Governance and funding. Shape decisions. Sponsor nodes for members who can't afford hardware.

**To join:** [Open an issue](https://github.com/meldfunction/kitchen-table/issues) titled "I want to get involved" and tell us what you're working with.

---

## Philosophy

We believe:
- **Tech should connect, not entrap.** The extractive model is a design choice, not a law of nature.
- **Cooperative governance produces better infrastructure** than corporate governance — for the people who use it.
- **The window is open.** Open models improve every 3–6 months. Hardware is cheap. Build now.
- **Local-first scales.** What works for one person's notes (Obsidian) works for a community's infrastructure.

We're inspired by Obsidian's local-first philosophy, Anytype's privacy-as-architecture approach, Elicit's honesty about AI capabilities, the Calm Tech Institute's attention-respecting design principles, and Patagonia's mission-as-governance model.

Full vision: [vision.html](https://meldfunction.github.io/kitchen-table/vision.html) · Full map: [map.html](https://meldfunction.github.io/kitchen-table/map.html)

---

## Contributing

PRs welcome. Open an issue before large changes to discuss direction.

Most valuable contributions right now:
- Setup guides for specific configurations (Windows host, macOS, specific Pi variants)
- Translations of getting-started documentation
- Corrections to any technical claims that are wrong or outdated
- Real-world configs from groups that have deployed this

See [CONTRIBUTING.md](CONTRIBUTING.md).

---

## License

MIT. Use it. Build on it. Share it. Make it better.

---

*Built April 2026. Open source. No ads. No VC. No quarterly earnings call.*
*Community governed · Local-first · The time is now.*
