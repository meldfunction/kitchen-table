# Exit Strategy — Full Provider Independence

## The honest starting point

You are probably dependent on some combination of:
- Anthropic / OpenAI / Google for AI inference
- Google / Apple for file storage and calendar
- Spotify / Apple Music for music
- Discord / WhatsApp / Telegram for group communication
- GitHub (Microsoft) for code
- Cloudflare / AWS for DNS and networking
- Google 8.8.8.8 for DNS resolution

Each dependency is a lever someone else holds. This document is about removing levers,
one at a time, in the order that delivers the most value with the least disruption.

You don't do this all at once. You do it in phases, over months. The goal is a
**sustainable exit**, not a dramatic one.

---

## Phase 1: AI inference independence (do this first)

**Current state:** API keys pointing at Anthropic, OpenAI, or Google.
**Target state:** All inference runs on your hardware using open-weight models.
**Time to complete:** 1 weekend.
**What you lose:** Frontier model quality on the hardest ~20% of tasks.
**What you gain:** Zero API costs, complete privacy, no rate limits, no terms of service.

Steps:
1. Install Ollama on your Pi (or any machine).
2. Pull Phi-3.5 Mini and Llama 3.2 1B.
3. Reconfigure OpenClaw to point at `http://127.0.0.1:11434/v1`.
4. Remove all API keys from your config.
5. Run `openclaw doctor` to confirm no cloud calls are being made.

The quality drop is real but smaller than expected. Phi-3.5 Mini handles most everyday
assistant tasks well. Use the collective Petals pool for harder tasks. Reserve any
remaining API keys for the specific cases where open models genuinely fail you —
and you'll find those cases are rarer than anticipated.

**Model quality progression over time:**
Open-weight models improve every 3–6 months. The gap between Llama 3.1 8B (2024) and
open models available in 2026 is significant. In 12 months it will be larger. Every
month you stay on open models, the tradeoff improves.

---

## Phase 2: File and data independence

**Current state:** Google Drive / Dropbox / iCloud.
**Target state:** NextCloud on your hardware, synced across the group.
**Time to complete:** 1–2 weekends.
**What you lose:** Seamless mobile sync (slightly worse UX), Google Docs real-time collaboration.
**What you gain:** Your files never leave hardware you control. No storage limits. No data mining.

Steps:
1. Install NextCloud AIO via CasaOS.
2. Install NextCloud desktop client on each person's machine.
3. Migrate your most important folder first (documents, not photos — photos come later).
4. Install Collabora Online for document editing (included in NextCloud AIO).
5. Migrate calendar and contacts to NextCloud (replaces Google Calendar / Apple iCloud).
6. After 2 weeks of normal use, migrate photos.

The migration strategy: don't export everything at once. Move what you actively use.
Old archives can stay where they are temporarily. The goal is stopping new data from
going to cloud providers, not necessarily recovering every old file immediately.

---

## Phase 3: Communication independence

**Current state:** Discord for group chat, WhatsApp for family.
**Target state:** Matrix homeserver for group, with bridges for people who won't switch.
**Time to complete:** 1 weekend for setup, 1–2 months for group adoption.
**What you lose:** Some Discord integrations, some WhatsApp features.
**What you gain:** End-to-end encryption on your hardware, no platform dependency.

The key insight: Matrix federates. Your friends on Matrix.org or any other homeserver
can talk to your homeserver. You don't need everyone to run their own server — just
interoperate with the broader Matrix network. Use bridges for people who refuse to
switch (Matrix bridges exist for WhatsApp, Telegram, Discord, Signal).

---

## Phase 4: Music independence

**Current state:** Spotify.
**Target state:** Navidrome serving your personal library to every device.
**Time to complete:** 1 afternoon.
**What you lose:** Spotify's discovery algorithm and curated playlists, podcasts.
**What you gain:** Lossless audio, no ads, no algorithm, music you actually own.

The gap: Spotify's discovery is genuinely good. You'll miss new music discovery.
Mitigations: RSS feeds for artist releases, last.fm integration with Navidrome,
sharing playlists between group members via Navidrome's built-in sharing.

For podcasts: AntennaPod (Android) or Overcast (iOS) with no backend required.
Podcasts are already decentralized (they're just RSS feeds). No self-hosting needed.

---

## Phase 5: Search independence

**Current state:** Google.
**Target state:** SearXNG instance on your hardware.
**Time to complete:** 30 minutes.
**What you lose:** Google's personalization (this is also what you're escaping).
**What you gain:** No search history, no profile building, no ads in results.

SearXNG queries Google, Bing, DuckDuckGo, and others simultaneously and returns
combined results — without sending your identity to any of them. Quality is close
to Google. The results are yours.

---

## Phase 6: Code hosting independence

**Current state:** GitHub.
**Target state:** Gitea self-hosted, with GitHub mirrors for public projects.
**Time to complete:** 1 afternoon.
**What you lose:** GitHub Actions CI/CD, GitHub's social features, discovery.
**What you gain:** Private repositories that are actually private, no Microsoft dependency.

Gitea runs on a Pi. It's a complete Git hosting platform: pull requests, issues,
wikis, project boards. For CI/CD: Woodpecker CI (Gitea-native) replaces GitHub Actions.

The migration approach: self-host Gitea for new private projects. Keep public projects
on GitHub (it's still the best place for open-source visibility). Mirror important
private repos from GitHub to Gitea. Don't rush the migration.

---

## Phase 7: DNS independence

**Current state:** ISP-provided DNS or Google 8.8.8.8.
**Target state:** Pi-hole + Unbound on your network.
**Time to complete:** 1–2 hours.
**What you gain:** Network-level ad blocking, no DNS queries going to Google,
  custom DNS entries for your Tailscale services.

```bash
# Pi-hole installation
curl -sSL https://install.pi-hole.net | bash

# Unbound (recursive DNS, no upstream provider)
sudo apt install unbound
# Configure Pi-hole to use Unbound as upstream
```

With Unbound, your DNS queries resolve directly against root servers — no Google,
no Cloudflare, no ISP snooping on what domains you visit.

---

## What you cannot fully replace (yet)

Be honest with yourself about the gaps:

**Frontier AI reasoning:** Open models are good. They are not GPT-4 or Claude Opus
for the hardest tasks. Keep an API key for the rare cases where you need it. The goal
is reducing dependency, not eliminating capability.

**Mobile hardware integration:** Apple and Google control the hardware your phone
runs on. You can replace most software. You cannot replace the operating system without
significant friction. GrapheneOS on a Pixel is the most complete mobile exit, but
it's a real commitment.

**Payments:** If you're paying for anything, you're dependent on Visa/Mastercard and
your bank. Cryptocurrency exists. It's not a complete solution. This is a genuine
unsolved problem.

**CDN / DDoS protection:** If you're running a public-facing service, you likely
need Cloudflare or similar. There are alternatives (BunnyCDN, Fastly), but this is
a real dependency for anything at scale.

---

## The philosophical endpoint

The goal isn't purity. Running Tailscale means trusting Tailscale. Running Matrix means
trusting the Matrix foundation. Using open-weight models means trusting that Meta
actually released the real weights.

The goal is **resilience**: reducing the number of single points of failure where
someone else's business decision, terms of service change, or server outage affects your
ability to function. The friend group stack described in this skill achieves that. It's
not a bunker. It's a more robust and private way to live digitally.

Every dependency you remove is one fewer lever. Remove the ones that cost you the most
in money, privacy, and autonomy first. The others can wait.
