# References

*Technical documentation, papers, and resources. Grouped by layer of the stack.*

---

## Inference and models

**llama.cpp documentation and build guides**
The canonical source for CPU inference. Build flags, quantization formats, and
performance tuning for different hardware.
→ https://github.com/ggerganov/llama.cpp/blob/master/docs/build.md

**Ollama model library**
Full list of models available via `ollama pull`. Each model card shows VRAM/RAM
requirements, parameter counts, and available quantizations.
→ https://ollama.com/library

**GGUF format specification**
The quantized model format used by llama.cpp and Ollama. Understanding it helps
you choose the right quantization (Q4_K_M vs Q8_0 etc.) for your RAM budget.
→ https://github.com/ggerganov/ggml/blob/master/docs/gguf.md

**Petals: Collaborative Inference and Fine-tuning of Large Models (paper)**
The academic paper behind the Petals distributed inference framework. Explains
the architecture, performance characteristics, and latency tradeoffs.
→ https://arxiv.org/abs/2209.01188

**Open LLM Leaderboard (Hugging Face)**
Benchmark comparisons across open-weight models. Use this to pick the best model
for your RAM budget — the leaderboard is sorted by performance, not by who
paid for the marketing.
→ https://huggingface.co/spaces/open-llm-leaderboard/open_llm_leaderboard

**TheBloke on Hugging Face**
The most prolific quantizer of open-weight models. If a model exists, TheBloke
probably has a GGUF version at every quantization level with benchmarks.
→ https://huggingface.co/TheBloke

**Unsloth (fine-tuning framework)**
Memory-efficient fine-tuning. Allows fine-tuning Llama 3.2 3B on hardware with
as little as 4GB RAM. The tool you use when you want to train a model on your
group's own data.
→ https://github.com/unslothai/unsloth

---

## Raspberry Pi and hardware

**Raspberry Pi 5 official documentation**
Complete hardware reference, GPIO pinout, NVMe HAT+ specs, power requirements.
→ https://www.raspberrypi.com/documentation/computers/raspberry-pi-5.html

**NVMe SSD boot guide for Pi 5**
Booting from NVMe (not SD card) is essential for inference workloads — SD card
I/O is a bottleneck. This guide covers the full process.
→ https://www.raspberrypi.com/documentation/computers/raspberry-pi-5.html#nvme-ssd-boot

**Jeff Geerling — Pi AI inference benchmarks**
Real benchmark data for various models on Pi 4 and Pi 5. The most honest
performance numbers available for this hardware.
→ https://www.jeffgeerling.com/blog/2023/pi-ai-benchmarks

**Compute Market — Home AI Server Guide 2026**
Hardware tier lists, GPU recommendations, and power consumption data for building
inference hardware at various budget levels.
→ https://www.compute-market.com/blog/home-ai-server-build-guide-2026

---

## Networking and mesh

**Tailscale documentation**
Complete reference for Tailscale setup, subnet routing, exit nodes, and ACLs.
The ACLs section is particularly important for multi-user group setups.
→ https://tailscale.com/kb/

**Tailscale + self-hosting guide**
Tailscale's own guide for securing self-hosted services. The right approach for
exposing NextCloud, Matrix, and other services to your group without public IPs.
→ https://tailscale.com/learn/self-hosting

**Yggdrasil Network documentation**
If you want a fully decentralized mesh (no Tailscale account required), Yggdrasil
is the alternative. End-to-end encrypted, no central key server.
→ https://yggdrasil-network.github.io

**Ansible for homelabs — getting started**
Practical guide to using Ansible to manage multiple Pi nodes. The pattern you
use to push config changes to your whole cluster in one command.
→ https://docs.ansible.com/ansible/latest/getting_started/

**WireGuard whitepaper**
The cryptographic protocol underlying Tailscale. Worth understanding if you're
making security decisions about your mesh.
→ https://www.wireguard.com/papers/wireguard.pdf

---

## Self-hosted services

**NextCloud documentation**
Complete admin and user docs. The Docker installation guide is the fastest path
for a new setup.
→ https://docs.nextcloud.com

**NextCloud AIO (All-in-One)**
Docker image that bundles NextCloud with Nginx, Collabora (online office),
and other dependencies. The recommended install for new setups.
→ https://github.com/nextcloud/all-in-one

**Navidrome documentation**
Setup, configuration, and Subsonic API reference. Includes Docker Compose examples.
→ https://www.navidrome.org/docs/

**Subsonic-compatible client list**
Apps for every platform that can stream from your Navidrome server.
→ https://www.navidrome.org/docs/overview/#apps

**Matrix homeserver setup (Synapse)**
The canonical Matrix server. Includes guides for federation (connecting to the
broader Matrix network) and island mode (private group only).
→ https://matrix-org.github.io/synapse/latest/setup/installation.html

**SearXNG documentation**
Installation, search engine configuration, and privacy settings.
→ https://docs.searxng.org

**CasaOS documentation**
App installation, storage management, and app store usage.
→ https://wiki.casaos.io

**Hive: A secure, scalable framework for distributed Ollama inference (paper)**
The academic paper describing the HiveCore/HiveNode architecture. Relevant if
you're evaluating Hive vs exo for your group's inference pooling.
→ https://www.sciencedirect.com/science/article/pii/S2352711025001505

---

## OpenClaw

**OpenClaw official documentation**
Architecture overview, configuration reference, gateway setup, and channel
integration guides.
→ https://github.com/openclaw/openclaw

**OpenClaw skills documentation**
How the SKILL.md format works, how skills are loaded, and how to write your own.
→ https://github.com/openclaw/openclaw/blob/main/docs/skills.md

**OpenClaw security advisories**
Current and historical CVEs. Subscribe to notifications if you're running a
gateway for a group.
→ https://github.com/openclaw/openclaw/security/advisories

**ClawHub (skill registry)**
The public skills marketplace. Check VirusTotal scan status before installing
anything. Only install from verified publishers.
→ https://clawhub.openclaw.dev

---

## Security and privacy

**OpenClaw security crisis retrospective**
The Reco AI blog post documenting the January–February 2026 vulnerability wave,
including CVE-2026-25253 and the ClawHavoc malicious skill campaign. Essential
reading before deploying a group gateway.
→ https://www.reco.ai/blog/openclaw-the-ai-agent-security-crisis-unfolding-right-now

**Prompt injection attacks on LLM applications**
Academic overview of prompt injection as an attack vector. Relevant for
understanding why OpenClaw skill vetting matters.
→ https://arxiv.org/abs/2302.12173

**Pi-hole documentation**
DNS-level ad and tracker blocking. The first thing to run on a group server.
→ https://docs.pi-hole.net

---

## Community knowledge bases

**Awesome-selfhosted**
A curated list of self-hosted services alternatives to every major SaaS product.
The authoritative reference for "what do I use instead of X."
→ https://github.com/awesome-selfhosted/awesome-selfhosted

**r/selfhosted wiki**
Community-maintained guides for common self-hosting setups.
→ https://www.reddit.com/r/selfhosted/wiki/

**LocalLLaMA wiki**
Community knowledge base for local model inference, hardware recommendations,
and model comparisons.
→ https://www.reddit.com/r/LocalLLaMA/wiki/

**Privacy Guides**
Recommendations for privacy-respecting alternatives to common software and services.
Useful companion when deciding which self-hosted services to prioritize.
→ https://www.privacyguides.org
