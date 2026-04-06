# Inspiration

*The people, projects, and ideas that proved owning your stack was possible.*

---

## The core idea

Every decade or so, the cost of compute drops enough that something previously only
possible at institutional scale becomes possible at the kitchen table. The 1970s gave
us personal computers. The 1990s gave us personal internet. The 2010s gave us personal
cloud storage. The 2020s are giving us personal AI and personal infrastructure.

The question is who captures the value. When compute is cheap and software is open,
the answer can be "you." When it's locked behind subscriptions and APIs, the answer
is someone else.

This stack is built on the belief that the infrastructure for a private, sovereign
digital life is already available. It just hasn't been assembled yet for most people.

---

## Projects that showed the way

**Raspberry Pi Foundation**
Proved that a $35 computer can run a serious workload. The Pi went from teaching kids
to code to anchoring home servers, mesh nodes, and now AI inference clusters. The Pi 5
with 8GB RAM running a quantized 3B model is a genuinely useful AI device.
→ https://www.raspberrypi.com

**Llama (Meta AI Research)**
When Meta released Llama weights openly in 2023, it changed everything. For the first
time, researchers and individuals could run frontier-class model weights on their own
hardware. Llama 3.x continued that trajectory — the 8B model is genuinely capable
for a wide range of tasks and runs on 8GB of RAM. The open-weight model movement
started here.
→ https://llama.meta.com

**llama.cpp (Georgi Gerganov)**
A C++ implementation of LLaMA inference optimized for CPU. Gerganov built something
that let people run AI on hardware they already had — no GPU required. The quantization
formats it pioneered (GGUF) became the standard for distributable model weights. This
is the software that made Pi inference real.
→ https://github.com/ggerganov/llama.cpp

**Petals (Hugging Face / BLOOM team)**
Demonstrated that large transformer models could be split across commodity hardware
over the internet. Not just locally — across nodes in different cities. If you can
run a shard, you contribute. The collective has more capability than any individual.
The research paper behind it ("Petals: Collaborative Inference and Fine-tuning of
Large Models") is worth reading.
→ https://petals.dev

**Tailscale**
Solved the hard part of running distributed services across home networks: the
networking. WireGuard under the hood, but with key management that just works.
A group of friends across different ISPs can have a private encrypted network in
20 minutes. The free tier is generous. The code is open.
→ https://tailscale.com

**NextCloud**
Proved that file sync, calendar, contacts, and collaborative documents don't require
Google or Microsoft. Started as a fork of ownCloud, now has a massive ecosystem of
apps. The fact that it's still growing while Google Drive exists is a testament to
how much people value owning their data.
→ https://nextcloud.com

**CasaOS (IceWhale Technology)**
Made self-hosting feel like using a smartphone. Before CasaOS, running a Docker app
meant editing YAML files and reading documentation. After CasaOS, it's a one-click
install from an app store. Lowered the barrier enough that non-technical family
members can manage their own home server apps.
→ https://casaos.io

**Matrix / Element**
Open-source, federated, end-to-end encrypted messaging. The protocol is what matters
— anyone can run a homeserver, anyone can build a client, and they all interoperate.
The fact that it bridges to Signal, Telegram, Discord, and IRC means your group
doesn't have to force everyone to switch at once.
→ https://matrix.org

**Navidrome**
A self-hosted music server so good it makes Spotify feel bloated. Streams to any
Subsonic-compatible client. Uses almost no resources. The founding insight: you
probably already own enough music. You just need a server.
→ https://www.navidrome.org

**SearXNG**
A meta-search engine that queries Google, Bing, DuckDuckGo, and dozens of others
but doesn't send identifying information and doesn't track you. Self-host it and
your group's searches stay on your hardware.
→ https://docs.searxng.org

**exo (ExoLabs)**
Demonstrated peer-to-peer distributed inference without a master-worker architecture.
Any device can contribute, any device can request. No central authority. The vision
of truly decentralized AI inference — this project is building it.
→ https://github.com/exo-explore/exo

**Hive (Domen Vake et al.)**
Published as an academic paper, implemented as open source. A framework for pooling
separate Ollama instances across different networks without requiring public ports.
Exactly the architecture a distributed friend group needs.
→ https://github.com/VakeDomen/HiveCore

---

## Communities worth knowing

**r/selfhosted**
The central community for people who run their own services. Enormous knowledge base
for everything from NextCloud setup to homelab networking. If you're stuck on something,
someone here has already solved it.
→ https://reddit.com/r/selfhosted

**r/LocalLLaMA**
The open-weight model community. Benchmark discussions, new model releases, hardware
recommendations, and real-world performance reports from people running models on
consumer hardware. Indispensable for staying current on which models to use.
→ https://reddit.com/r/LocalLLaMA

**r/homelab**
More infrastructure-focused than r/selfhosted. Hardware builds, networking, server
setups. Where you go when your Pi cluster graduates to something bigger.
→ https://reddit.com/r/homelab

**r/OpenClawUseCases**
Real configs, deployment patterns, and agent setups from the OpenClaw community.
Worth monitoring for patterns that work in production.
→ https://reddit.com/r/OpenClawUseCases

**Hacker News**
Not a community exactly, but the comment threads on self-hosting and open-source AI
stories surface some of the best technical thinking anywhere. Search for "local LLM"
or "self-hosted" in HN archives for dense technical discussion.
→ https://news.ycombinator.com

---

## People doing it in the open

**Jeff Geerling (@geerlingguy)**
Documents everything he builds, including Pi clusters, Mac Mini clusters for AI
inference, and homelab networking. His YouTube channel is the closest thing to a
structured curriculum for this space.
→ https://www.jeffgeerling.com

**Andrej Karpathy**
Periodically demonstrates that frontier AI concepts are understandable and reproducible.
His "build a ChatGPT clone for $15–100" experiments established the ceiling on what
commodity compute can do. Follow for the most grounded thinking on open AI.
→ https://karpathy.ai

**Simon Willison**
Prolific writer on open-source LLMs, local inference, and practical AI tooling.
His blog has documented every significant development in open-weight models since
Llama 1. Excellent signal-to-noise ratio.
→ https://simonwillison.net

---

## The longer arc

The current moment — where you can run a capable AI model on a $80 computer, sync
files across a friend group for free, and communicate without feeding a surveillance
platform — is historically unusual. It exists because enough people chose to build
open alternatives instead of just consuming proprietary ones.

The stack in this guide builds on a decade of that work. The least you can do is
use it, maintain it, and eventually contribute back.

The people who are still self-hosting five years from now will have started now.
