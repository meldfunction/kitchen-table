# The Sovereign Stack
## A complete guide to self-hosted AI and services for friend groups

*Built from the open-claw-net and open-claw skills · April 2026*

---

> "Every SaaS subscription your group pays is rent on infrastructure someone else controls.
> Every API key is a dependency on a company whose terms, pricing, and existence you cannot
> guarantee. The barrier to owning your stack is knowledge, not money. This guide is the knowledge."

This guide covers two things that belong together but are rarely explained together:

1. **OpenClaw** — the open-source AI agent platform you configure, extend, and run yourself
2. **The sovereign stack** — the complete infrastructure for replacing cloud AI and SaaS apps
   with hardware you own, software you control, and a private mesh network

You can read it front to back as a walkthrough, or jump to any section using the table of
contents below. Every code block is ready to run. Every configuration is explained, not just
listed.

---


## Table of contents

### Part 1 — OpenClaw basics
- [1.1 What OpenClaw actually is](#11-what-openclaw-actually-is)
- [1.2 CLAUDE.md — the highest-leverage config](#12-claudemd--the-highest-leverage-config)
- [1.3 Permissions — eliminate click fatigue](#13-permissions--eliminate-click-fatigue)
- [1.4 Session management](#14-session-management)
- [1.5 Parallel subagents](#15-parallel-subagents)
- [1.6 Hooks — the real extension API](#16-hooks--the-real-extension-api)
- [1.7 MCP servers](#17-mcp-servers)
- [1.8 Interruption and retry](#18-interruption-and-retry)

### Part 2 — Advanced OpenClaw configuration
- [2.1 Advanced CLAUDE.md patterns](#21-advanced-claudemd-patterns)
- [2.2 Advanced hooks recipes](#22-advanced-hooks-recipes)

### Part 3 — The sovereign stack (Pi-first, no GPUs, no paid APIs)
- [3.1 Philosophy and architecture choice](#31-philosophy-and-architecture-choice)
- [3.2 Hardware — how many Pis and what kind](#32-hardware--how-many-pis-and-what-kind)
- [3.3 Software prerequisites per person](#33-software-prerequisites-per-person)
- [3.4 Mesh network with Tailscale](#34-mesh-network-with-tailscale)
- [3.5 Inference stack — Ollama, llama.cpp, Petals](#35-inference-stack--ollama-llamacpp-petals)
- [3.6 OpenClaw gateway — local models only](#36-openclaw-gateway--local-models-only)
- [3.7 Multi-user access and workspace isolation](#37-multi-user-access-and-workspace-isolation)
- [3.8 SOUL.md — identity and boundaries](#38-soulmd--identity-and-boundaries)

### Part 4 — Shared services (replacing SaaS)
- [4.1 CasaOS — your app control plane](#41-casaos--your-app-control-plane)
- [4.2 NextCloud — files, calendar, photos](#42-nextcloud--files-calendar-photos)
- [4.3 Navidrome — music streaming](#43-navidrome--music-streaming)
- [4.4 Matrix + Element — communication](#44-matrix--element--communication)
- [4.5 SearXNG, Gitea, Vaultwarden, Pi-hole](#45-searxng-gitea-vaultwarden-pi-hole)

### Part 5 — Open models and getting off the provider teat
- [5.1 Which models to trust and why](#51-which-models-to-trust-and-why)
- [5.2 Fine-tuning your own model on group data](#52-fine-tuning-your-own-model-on-group-data)
- [5.3 The exit strategy — provider independence roadmap](#53-the-exit-strategy--provider-independence-roadmap)

### Part 6 — Mesh networking deep dive
- [6.1 Tailscale ACLs and subnet routing](#61-tailscale-acls-and-subnet-routing)
- [6.2 Ansible — pushing configs to all nodes at once](#62-ansible--pushing-configs-to-all-nodes-at-once)
- [6.3 Yggdrasil — the fully decentralised alternative](#63-yggdrasil--the-fully-decentralised-alternative)

### Part 7 — Inspiration and references
- [7.1 Inspiration — projects and people](#71-inspiration--projects-and-people)
- [7.2 Technical references](#72-technical-references)

### Appendix
- [Quick start checklist](#quick-start-checklist)
- [Cost breakdown](#cost-breakdown)
- [Security checklist](#security-checklist)
- [Migration from Clawdbot / Moltbot](#migration-from-clawdbot--moltbot)

---


---

# Part 1 — OpenClaw — what it actually is and how to configure it
*Based on source-code analysis of the Claude Code / OpenClaw architecture*


# Open Claw — Claude Code Setup Guide

Claude Code (nicknamed "Claw") is an agent orchestration platform wearing a terminal UI. Most
people use it like a chatbot. This skill helps configure it as the power tool it actually is.

Work through the sections below in order for a first-time setup, or jump to any section if the
user needs help with a specific area.

---

## 1. CLAUDE.md — The Highest-Leverage Config

**Why it matters:** Claude Code re-reads ALL CLAUDE.md files on every single query — not just at
session start. This means it's a persistent, always-active system prompt you control completely.

### File hierarchy

| File | Scope | Use for |
|---|---|---|
| `~/.claude/CLAUDE.md` | Global | Coding style, personal preferences, general rules |
| `./CLAUDE.md` | Project root | Architecture decisions, conventions, tech stack |
| `.claude/rules/*.md` | Modular rules | Per-domain rules (e.g., `testing.md`, `security.md`) |
| `CLAUDE.local.md` | Private (gitignored) | Personal notes, local secrets, dev environment quirks |

**Limit:** 40,000 characters. Most people use ~200. Fill it up.

### What to put in CLAUDE.md

Write instructions as if briefing a new senior engineer on your project every single time they
open a file. Include:

- **Architecture overview** — What the system does, major components, data flow
- **File/folder conventions** — Where things live and why
- **Tech stack specifics** — Versions, libraries, patterns in use
- **Testing patterns** — How tests are structured, how to run them, what coverage is expected
- **Never-do rules** — Anti-patterns, banned approaches, things that have caused bugs before
- **Git conventions** — Branch naming, commit message format, PR expectations
- **External dependencies** — APIs, services, environment variables required

### Template for a project CLAUDE.md

```markdown
# Project: [Name]

## What this does
[2-3 sentence summary of the system's purpose]

## Architecture
[Key components and how they relate]

## Tech stack
- Language: 
- Framework: 
- Database: 
- Key libraries: 

## File conventions
- `src/` — [what goes here]
- `tests/` — [structure and naming]
- `scripts/` — [what goes here]

## Testing
- Run tests with: `[command]`
- Test files named: `[pattern]`
- Mocking approach: [describe]

## NEVER do this
- [Anti-pattern 1 and why]
- [Anti-pattern 2 and why]

## Commit format
[Your convention here]
```

For detailed guidance on advanced CLAUDE.md patterns, see `references/claude-md-advanced.md`.

---

## 2. Permission Configuration — Eliminate Click Fatigue

Every time Claude Code asks "Allow this action?" and you click yes, that's a missed configuration
opportunity. Set it once, never click again.

### Settings file location

```
~/.claude/settings.json   ← user-level (your machine)
.claude/settings.json     ← project-level (committed, shared with team)
```

### Recommended starter config

```json
{
  "permissions": {
    "allow": [
      "Bash(npm *)",
      "Bash(yarn *)",
      "Bash(pnpm *)",
      "Bash(git *)",
      "Bash(python *)",
      "Bash(pytest *)",
      "Edit(src/**)",
      "Write(src/**)",
      "Edit(tests/**)",
      "Write(tests/**)"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Bash(curl * | bash *)"
    ]
  }
}
```

### Permission modes

| Mode | Behavior | When to use |
|---|---|---|
| `auto` | LLM classifier evaluates each action against your allow/deny lists | **Recommended default** |
| `allowEdits` | Auto-allows all file edits in working directory | Fast iteration on your own code |
| `bypass` | No permission checks at all | Trusted automation pipelines only |

Set the mode in `settings.json`:
```json
{ "permissionMode": "auto" }
```

---

## 3. Session Management — Stop Starting Fresh

Every new session loses accumulated context. Use persistence.

### Key flags

```bash
claude --continue          # Resume the last session
claude --resume            # Pick a specific past session from a list
claude --fork-session      # Branch from a past session (great for experiments)
```

Sessions are stored as JSONL at:
```
~/.claude/projects/{project-hash}/{sessionId}.jsonl
```

### The /compact command

Context pressure is real. Claude Code has 5 compaction strategies internally, but `/compact` is
your manual control. Think of it like a save point:

- Use it **before** a major new task within the same session
- Use it when you've finished a large feature and are starting something new
- Use it if responses start feeling less precise (context noise building up)

For large refactors: use the `[1m]` model suffix to opt into the 1M token context window.

**Default workflow:**
```
claude --continue          # always start by resuming
/compact                   # compact before big new tasks
```

---

## 4. Parallel Subagents — 5 Agents ≈ Cost of 1

When Claude Code forks a subagent, it creates a byte-identical copy of the parent context. The
API caches this shared context. Spawning 5 agents costs barely more than 1 because they all
hit the prompt cache.

### Three subagent execution models

| Model | How it works | Best for |
|---|---|---|
| `fork` | Inherits parent context, cache-optimized | Parallel tasks on the same codebase |
| `teammate` | Separate pane (tmux/iTerm), file-based mailbox | Long-running parallel workstreams |
| `worktree` | Own git worktree, isolated branch | Risky changes, experiments, A/B work |

### Example: Parallel workstreams

Tell Claude Code:
```
Spin up 4 agents in parallel:
1. Security audit of the auth module
2. Refactor the database layer to use the repository pattern
3. Write tests for all untested utilities in src/utils/
4. Update the API documentation in docs/
```

All 4 run simultaneously. All hit the same prompt cache. Output costs the same per-agent, but
input token costs are negligible thanks to caching.

---

## 5. Hooks — The Real Extension API

Hooks let you inject behavior at 25+ lifecycle points without modifying prompts. This is how
you build a custom development environment on top of Claude Code.

### Five hook types

| Type | What it does |
|---|---|
| `command` | Run a shell command |
| `prompt` | Inject context via an LLM call |
| `agent` | Run a full agent verification loop |
| `HTTP` | Call a webhook |
| `function` | Run JavaScript |

### Key lifecycle events

| Event | Fires when |
|---|---|
| `PreToolUse` | Before any tool executes |
| `PostToolUse` | After any tool executes |
| `UserPromptSubmit` | When you send a message |
| `SessionStart` | Session begins |
| `SessionEnd` | Session ends |

### Hook config location

```
.claude/hooks/
├── pre-tool-use.json
├── post-tool-use.json
└── user-prompt-submit.json
```

### High-value hook recipes

**Auto-lint before file writes:**
```json
{
  "event": "PreToolUse",
  "tool": "Write",
  "hook": {
    "type": "command",
    "command": "eslint {{file}} --fix"
  }
}
```

**Auto-run tests after edits:**
```json
{
  "event": "PostToolUse",
  "tool": "Edit",
  "hook": {
    "type": "command",
    "command": "npm test -- --testPathPattern={{file}}"
  }
}
```

**Inject context into every prompt (UserPromptSubmit):**
```json
{
  "event": "UserPromptSubmit",
  "hook": {
    "type": "prompt",
    "additionalContext": "Current git diff: $(git diff --stat HEAD)"
  }
}
```

For more hook patterns and advanced configuration, see `references/hooks-advanced.md`.

---

## 6. MCP Servers — Extend the Tool System

Claude Code has 60+ built-in tools. MCP servers add more. They use deferred loading — connecting
5 MCP servers doesn't slow anything down because tools only load when actually used.

### Common MCP servers worth connecting

| Server | Adds |
|---|---|
| `@anthropic/mcp-server-filesystem` | Enhanced file operations |
| `@modelcontextprotocol/server-github` | GitHub API tools |
| `@modelcontextprotocol/server-postgres` | Direct DB queries |
| `@modelcontextprotocol/server-puppeteer` | Browser automation |

### Connect an MCP server

```bash
claude mcp add <server-name> <command>
# Example:
claude mcp add github npx @modelcontextprotocol/server-github
```

Rule of thumb: if your workflow touches an external system (CI/CD, cloud provider, database,
issue tracker), there's likely an MCP server for it. Connect it and Claude Code can interact
with it directly.

---

## 7. Interruption — Zero Penalty for Redirecting

The entire streaming pipeline uses async generators. Pressing `Escape` cleanly aborts the current
stream without losing previous context. The interrupted response is discarded; your history is
intact.

**Use this aggressively.** If Claude Code starts going the wrong direction:
- Don't wait for it to finish
- Hit `Escape` immediately
- Redirect with a corrected prompt

This is pair programming behavior. A good pair doesn't let their partner go 5 minutes down the
wrong path before saying something.

---

## Quick Setup Checklist

Run through this for any new project:

- [ ] Create `~/.claude/CLAUDE.md` with your global preferences and coding style
- [ ] Create `./CLAUDE.md` with project architecture, conventions, and never-dos
- [ ] Create `.claude/settings.json` with permission allow-list for your common commands
- [ ] Set `"permissionMode": "auto"` in settings
- [ ] Test with `claude --continue` as your default way to open Claude Code
- [ ] Identify 1-2 hooks that would save you repetitive work and configure them
- [ ] Connect any MCP servers relevant to your external tools
- [ ] Practice `/compact` before switching to a new major task

---

## Reference Files

- `references/claude-md-advanced.md` — Advanced CLAUDE.md patterns, examples from real projects
- `references/hooks-advanced.md` — Full hook event list, advanced hook recipes, debugging hooks


---

> **Where we're going next:** The sections above covered OpenClaw's core mechanics when pointed
> at a cloud model. Part 2 goes deeper — advanced CLAUDE.md patterns and hook recipes that
> make OpenClaw feel like a custom development environment rather than a generic assistant.
> After that, Part 3 throws away the API keys entirely.

---


---

# Part 2 — Advanced OpenClaw configuration
*Deep patterns for teams who want OpenClaw to feel like a custom environment*


## 2.1 Advanced CLAUDE.md patterns


# Advanced CLAUDE.md Patterns

## Real-world examples and patterns for power users

---

## Pattern: Technology-specific rules

Break out rules by concern using the `.claude/rules/` directory. Claude Code loads these
alongside your main CLAUDE.md on every turn.

```
.claude/rules/
├── testing.md        ← testing conventions
├── security.md       ← security requirements
├── api-design.md     ← REST/GraphQL patterns
└── accessibility.md  ← a11y requirements
```

Each file is focused and can be maintained independently by different team members.

---

## Pattern: Explicit anti-patterns section

The "NEVER do this" section is often the highest-signal content in a CLAUDE.md. Be specific
about why — it helps the model understand the intent, not just the rule.

```markdown
## Never do this

- **Never use `any` in TypeScript** — defeats type safety, causes silent bugs downstream
- **Never commit .env files** — even to branches; rotate secrets if this happens
- **Never use `console.log` for debug output** — use the `logger` utility with log levels
- **Never mutate props in React components** — creates unpredictable re-render behavior
- **Never write raw SQL** — use the QueryBuilder in `src/db/query-builder.ts`
```

---

## Pattern: Inline architecture diagram

ASCII or text diagrams in CLAUDE.md help the model understand data flow and component
relationships without needing to read every file.

```markdown
## System architecture

Request flow:
  Client → API Gateway → Auth Middleware → Route Handler → Service Layer → Repository → DB

Key services:
  UserService ← manages auth and profiles
  OrderService ← handles all transaction logic (depends on UserService, PaymentService)
  PaymentService ← wraps Stripe, never called directly by route handlers

Rule: Services only call other services, never repositories from other domains.
```

---

## Pattern: Environment and dev setup

Tell Claude Code what your local environment looks like so it doesn't assume defaults.

```markdown
## Dev environment

- Node: 20.x (use nvm, .nvmrc is committed)
- Package manager: pnpm (not npm or yarn)
- Database: Postgres 15, running on localhost:5432, DB name: myapp_dev
- Redis: localhost:6379 (required for session storage and queues)
- Env file: .env.local (copy from .env.example, never commit)

To start dev: `pnpm dev` (starts API on :3001, web on :3000)
To run migrations: `pnpm db:migrate`
```

---

## Pattern: Team-specific conventions

Use CLAUDE.md to encode decisions that are specific to your team — things that wouldn't be
obvious from reading the code.

```markdown
## Team conventions

- PRs: max 400 lines changed. Split larger changes.
- Branch naming: `{type}/{ticket-id}-{short-description}` (e.g., `feat/ENG-123-user-profiles`)
- Commit messages: Conventional Commits format (feat/fix/chore/docs/refactor)
- Code review: at least 1 approval required, author merges
- Feature flags: use the `flags` module in `src/config/flags.ts` for all new features

When in doubt, ask in #eng-standards Slack before deviating from these.
```

---

## Pattern: CLAUDE.local.md for personal notes

CLAUDE.local.md is gitignored — it's for things that are true for you but not the whole team.

```markdown
# My local notes

- I have a staging DB tunnel running at localhost:5433
- My test user is test@example.com / password123
- I'm currently focused on the payments module, ignore auth for now
- The mobile team's staging URL is https://staging-mobile.example.com
```

---

## Sizing guidance

| Project size | Approximate CLAUDE.md length |
|---|---|
| Solo side project | 500–1,000 chars |
| Small team (2-5) | 2,000–5,000 chars |
| Medium team (5-20) | 5,000–15,000 chars |
| Large codebase | 15,000–40,000 chars (use modular rules/) |

The 40K character limit is generous. If you're under 2,000 chars on a real project, you're
probably leaving guidance on the table.

---

## Updating CLAUDE.md as you work

Good practice: when Claude Code makes a decision you want to preserve (a pattern it invented,
a convention it settled on, a bug fix approach), add it to CLAUDE.md immediately. The model
will incorporate that decision into all future turns automatically.

Think of it as a living document — not a one-time setup artifact.


---


## 2.2 Advanced hooks recipes


# Advanced Hooks Patterns

## Full lifecycle event reference

| Event | Fires when | Common use |
|---|---|---|
| `PreToolUse` | Before any tool call | Validate, lint, gate |
| `PostToolUse` | After any tool call | Test, notify, log |
| `UserPromptSubmit` | User sends a message | Inject context |
| `SessionStart` | Session begins | Load state, greet |
| `SessionEnd` | Session ends | Save state, notify |
| `ModelResponse` | Model responds | Post-process output |
| `ToolError` | A tool throws | Alert, retry logic |
| `MemorySearch` | Memory is queried | Augment results |
| `CronFire` | Scheduled job triggers | Pre-flight checks |
| `ChannelMessage` | Inbound from any channel | Pre-process messages |

---

## Hook type reference

### command hook
Runs a shell command. Use for linting, testing, git ops.

```json
{
  "event": "PostToolUse",
  "tool": "Edit",
  "hook": {
    "type": "command",
    "command": "npm test -- --testPathPattern={{file}} --passWithNoTests",
    "timeout": 30000
  }
}
```

### prompt hook
Calls an LLM to inject context or validate. Use for semantic checks.

```json
{
  "event": "PreToolUse",
  "tool": "Write",
  "hook": {
    "type": "prompt",
    "prompt": "Does this file write follow our security conventions? Output APPROVE or REJECT with reason.",
    "onReject": "abort"
  }
}
```

### agent hook
Spawns a full agent loop for complex verification. Use sparingly — it's expensive.

```json
{
  "event": "PostToolUse",
  "tool": "Bash",
  "hook": {
    "type": "agent",
    "task": "Verify the last bash command didn't modify files outside the src/ directory."
  }
}
```

### HTTP hook
Calls a webhook. Use for notifications, external triggers.

```json
{
  "event": "SessionEnd",
  "hook": {
    "type": "HTTP",
    "url": "https://hooks.slack.com/services/YOUR/WEBHOOK/URL",
    "method": "POST",
    "body": {
      "text": "OpenClaw session completed: {{sessionSummary}}"
    }
  }
}
```

### function hook
Runs JavaScript inline. Use for data transformation or conditional logic.

```json
{
  "event": "UserPromptSubmit",
  "hook": {
    "type": "function",
    "code": "return { additionalContext: `Git status: ${await exec('git status --short')}` };"
  }
}
```

---

## High-value hook recipes

### Auto-inject git diff into every prompt
Never manually paste `git diff` again:

```json
{
  "event": "UserPromptSubmit",
  "hook": {
    "type": "function",
    "code": "const diff = await exec('git diff --stat HEAD'); return { additionalContext: `Recent changes:\\n${diff}` };"
  }
}
```

### Auto-run tests after any file edit
Catch broken tests immediately without asking:

```json
{
  "event": "PostToolUse",
  "tool": "Edit",
  "hook": {
    "type": "command",
    "command": "npm test -- --passWithNoTests 2>&1 | tail -20",
    "timeout": 60000
  }
}
```

### Slack notification on session end
Know when a long-running task completes:

```json
{
  "event": "SessionEnd",
  "hook": {
    "type": "HTTP",
    "url": "{{SLACK_WEBHOOK_URL}}",
    "method": "POST",
    "body": { "text": "✅ OpenClaw task complete — check your terminal." }
  }
}
```

### Security gate before file writes
Block writes outside allowed directories:

```json
{
  "event": "PreToolUse",
  "tool": "Write",
  "hook": {
    "type": "function",
    "code": "if (!args.path.startsWith('src/') && !args.path.startsWith('tests/')) { return { abort: true, reason: 'Writes only allowed in src/ and tests/' }; }"
  }
}
```

---

## Debugging hooks

- Check the OpenClaw logs at `~/.openclaw/logs/` for hook execution output
- Use `openclaw doctor` to validate hook config syntax
- Add `"debug": true` to any hook to get verbose execution logs


---

> **The pivot.** Everything in Parts 1 and 2 assumed you had an Anthropic or OpenAI API key.
> Part 3 is built on the premise that you shouldn't need one — and explains exactly how to
> run the same workflows on hardware you own, with models you can download and run offline,
> across a private network your group controls.

---


---

# Part 3 — The sovereign stack — Pi-first, no GPUs, no paid APIs
*A complete setup guide for a friend group's decentralised AI cluster*


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


---

> **Inference is only half the stack.** Once your group has AI running on local hardware,
> the natural next step is replacing the other services you're renting. Part 4 covers the
> full SaaS replacement map — music, files, communication, search, code hosting, passwords,
> and DNS — all running on the same Pi infrastructure.

---


---

# Part 4 — Shared services — replacing SaaS
*The full replacement map: Spotify → Navidrome, Google Drive → NextCloud, Discord → Matrix*


# Self-Hosted Services — Complete Stack Reference

## The SaaS replacement map

| You pay for | You replace it with | Self-hosted app | Difficulty |
|---|---|---|---|
| Spotify | Your music library | Navidrome | Easy |
| Google Drive | Your files | NextCloud | Medium |
| Google Docs | Collaborative editing | NextCloud + Collabora | Medium |
| Google Calendar | Calendar sync | NextCloud Calendar | Easy |
| Google Contacts | Contact sync | NextCloud Contacts | Easy |
| Google Photos | Photo backup | NextCloud Photos or Immich | Medium |
| Discord | Group chat | Matrix + Element | Medium |
| WhatsApp | Messaging | Matrix (with bridges) | Medium |
| Dropbox | File sync | NextCloud | Medium |
| GitHub (private) | Code hosting | Gitea | Medium |
| Notion | Notes + docs | Outline or Joplin Server | Medium |
| Pocket/Instapaper | Read-later | Wallabag | Easy |
| Google Search | Web search | SearXNG | Easy |
| Cloudflare 1.1.1.1 DNS | DNS resolution | Pi-hole + Unbound | Medium |
| Bitwarden (paid) | Password manager | Vaultwarden | Easy |

---

## CasaOS — your app control plane

CasaOS runs on any machine with Docker. Install it once and manage all services from
a browser UI. Every app below can be installed via CasaOS's app store in 2 clicks.

```bash
curl -fsSL https://get.casaos.io | sudo bash
```

Access CasaOS at `http://<your-node-tailscale-ip>:80`

CasaOS handles: Docker Compose, persistent storage, automatic restarts, update
notifications, and resource monitoring. You rarely need to touch YAML files.

---

## NextCloud — files, calendar, contacts, photos

### Installation (NextCloud AIO recommended)

```bash
# Via CasaOS: Apps → Community Apps → NextCloud AIO
# Or manually:
docker run -d \
  --name nextcloud-aio-mastercontainer \
  --restart always \
  -p 8080:8080 \
  -e APACHE_PORT=11000 \
  -v nextcloud_aio_mastercontainer:/mnt/docker-aio-config \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  nextcloud/all-in-one:latest
```

Access setup at `https://<tailscale-ip>:8080`

NextCloud AIO includes: NextCloud core, Collabora Online (document editing),
Nextcloud Talk (video calls), ClamAV (virus scanning), Imaginary (image processing),
Fulltextsearch (file search). Everything in one Docker stack.

### Client apps per person

| Platform | App | Notes |
|---|---|---|
| Windows / macOS | NextCloud Desktop | Official, works well |
| Android | NextCloud app | File sync + auto photo upload |
| iOS | NextCloud app | Same |
| Calendar (any) | Any CalDAV client | Built into iOS/macOS/Thunderbird |
| Contacts (any) | Any CardDAV client | Built into iOS/macOS/Thunderbird |

### Photo backup setup

Configure each person's phone to auto-upload photos to NextCloud. Photos stay on
your hardware, not Google Photos or iCloud. The NextCloud iOS/Android apps have
auto-upload built in.

```
NextCloud Settings → Mobile devices → Auto-upload
Set upload folder: /Photos/<your-name>/
Enable: Upload on WiFi only (saves data)
```

---

## Navidrome — music streaming (Spotify replacement)

### What it is

Navidrome indexes your music library and serves it via the Subsonic API. Any Subsonic-
compatible app becomes your Spotify alternative. The group's collective library is
available to everyone.

### Installation

```yaml
# docker-compose.yml
services:
  navidrome:
    image: deluan/navidrome:latest
    restart: unless-stopped
    ports:
      - "4533:4533"
    volumes:
      - /your/music/library:/music:ro
      - navidrome_data:/data
    environment:
      ND_SCANSCHEDULE: "1h"
      ND_LOGLEVEL: "info"
      ND_SESSIONTIMEOUT: "24h"
      ND_BASEURL: ""
```

### Music library organization

Everyone contributes to the shared music folder. A simple convention:

```
/music/
├── library/
│   ├── Artist Name/
│   │   ├── Album Name (Year)/
│   │   │   ├── 01 - Track Name.flac
│   │   │   └── cover.jpg
```

Use beets (https://beets.io) to auto-organize and tag your existing library before
importing. Running `beet import /your/music` will sort everything into the right
structure automatically.

### Client apps

| Platform | App | Free? |
|---|---|---|
| Android | Symfonium | Paid (~$5, worth it) |
| Android | Subsonic (DSub) | Free |
| iOS | Amperfy | Free |
| iOS | play:Sub | Paid |
| Desktop | Feishin | Free, open source |
| Web | Navidrome web UI | Built-in |

### Playlist sharing

Navidrome has a built-in playlist system. Create a shared playlist, add it to a
shared account, and everyone in the group can follow it. Works like Spotify's
collaborative playlists.

---

## Matrix / Element — group communication

### Why Matrix over Signal or Telegram

Signal: excellent encryption, but centralized. Signal's servers or phone number
requirement can be a single point of failure.

Telegram: not end-to-end encrypted by default, centralized Russian company.

Matrix: federated protocol. Your homeserver talks to any other Matrix server.
Even if your server goes down, everyone's messages are preserved on their own
clients. Bridges exist for Signal, Telegram, WhatsApp, Discord.

### Synapse homeserver installation

```bash
# Generate config
docker run -it --rm \
  -v ~/synapse:/data \
  -e SYNAPSE_SERVER_NAME=matrix.your-tailscale-hostname \
  -e SYNAPSE_REPORT_STATS=no \
  matrixdotorg/synapse:latest generate

# Run
docker run -d \
  --name synapse \
  --restart unless-stopped \
  -p 8448:8448 \
  -v ~/synapse:/data \
  matrixdotorg/synapse:latest
```

### Client apps

| Platform | App | Notes |
|---|---|---|
| All platforms | Element | Official, feature-complete |
| Android | Schildichat | Element fork, better UX |
| iOS | Element X | New faster client |
| Desktop | Cinny | Clean web-based client |
| Desktop | Beeper | Aggregates multiple platforms |

### Bridges for people who won't switch

Matrix bridges let your homeserver relay messages to/from other platforms.
The person on Matrix talks to the person on WhatsApp without either switching apps.

Popular bridges:
- mautrix-whatsapp: WhatsApp bridge
- mautrix-telegram: Telegram bridge
- mautrix-signal: Signal bridge  
- matrix-appservice-discord: Discord bridge

---

## Immich — Google Photos replacement

Navidrome is for music. Immich is for photos. It has face recognition, location
mapping, album sharing, and mobile apps that auto-backup your camera roll.

```yaml
services:
  immich-server:
    image: ghcr.io/immich-app/immich-server:release
    restart: always
    ports:
      - "2283:2283"
    volumes:
      - /your/photos:/usr/src/app/upload
      - /etc/localtime:/etc/localtime:ro
```

---

## Vaultwarden — password manager (Bitwarden compatible)

Self-hosted Bitwarden server. Use any Bitwarden client app. Your group's passwords
stay on your hardware.

```bash
docker run -d \
  --name vaultwarden \
  -p 8082:80 \
  -v vaultwarden_data:/data \
  vaultwarden/server:latest
```

Point the Bitwarden app to your Tailscale IP instead of bitwarden.com.

---

## Gitea — GitHub replacement

```bash
docker run -d \
  --name gitea \
  -p 3000:3000 \
  -p 222:22 \
  -v gitea_data:/data \
  gitea/gitea:latest
```

Gitea UI is almost identical to GitHub. Includes pull requests, issues, wikis,
project boards, and CI/CD via Woodpecker CI (a separate service that integrates
natively with Gitea).

---

## Pi-hole + Unbound — DNS independence

Pi-hole blocks ads and trackers at the network level. Unbound resolves DNS directly
from root servers without sending queries to Google or Cloudflare.

```bash
# Pi-hole
curl -sSL https://install.pi-hole.net | bash

# Unbound
sudo apt install unbound

# Configure Pi-hole to use Unbound as upstream (127.0.0.1#5335)
# See: https://docs.pi-hole.net/guides/dns/unbound/
```

Set Pi-hole as the DNS server in your router (or on each device) and every device
on your Tailscale network gets ad blocking and private DNS.


---

> **The deeper question.** Parts 3 and 4 cover infrastructure and services. Part 5 is about
> the models themselves — which open-weight models to trust, how their licenses actually work,
> how to fine-tune a small model on your group's own data, and what a realistic provider
> exit roadmap looks like over 6–12 months.

---


---

# Part 5 — Open models and getting off the provider teat
*Which models to trust, how to fine-tune your own, and a realistic independence roadmap*


## 5.1 Which models to trust and why


# Open Models — Selection, Licenses, and Fine-tuning

## The rule: if you can't download it and run it offline, it's not open

"Open" AI is a marketing term. For this stack, open means:
1. Model weights are publicly downloadable
2. You can run inference without an internet connection
3. The license permits your use case (personal + small group)

---

## Recommended models by use case (Pi 5, 8GB RAM)

### Personal agent / everyday assistant

**Phi-3.5 Mini Instruct (3.8B, Q4_K_M)**
Best reasoning-per-RAM-dollar in its class. Handles coding, writing, analysis, and
planning well. Microsoft's training choices for this model size are genuinely impressive.
License: MIT (fully open, commercial use permitted)
Pull: `ollama pull phi3.5:3.8b-mini-instruct-q4_K_M`
RAM: ~2.4GB | Speed: ~3–5 tok/s on Pi 5

**Llama 3.2 3B Instruct (Q4_K_M)**
Meta's small model, excellent instruction following. Slightly behind Phi-3.5 on
reasoning but strong on conversational tasks.
License: Llama 3.2 Community License (free for most uses under 700M MAU)
Pull: `ollama pull llama3.2:3b-instruct-q4_K_M`
RAM: ~2.0GB | Speed: ~4–6 tok/s on Pi 5

### Background tasks / heartbeats / subagents

**Llama 3.2 1B Instruct (Q8_0)**
Use this for everything OpenClaw runs in the background. It's fast enough for
heartbeats, light enough to not crowd out your main model, and smart enough for
simple classification and routing tasks.
Pull: `ollama pull llama3.2:1b-instruct-q8_0`
RAM: ~1.3GB | Speed: ~8–12 tok/s on Pi 5

### Coding tasks

**Qwen2.5 Coder 3B (Q4_K_M)**
Purpose-built for code. Outperforms larger general models on coding benchmarks at
this parameter count. If your group does software development, this is your coding
assistant.
License: Apache 2.0 (fully open)
Pull: `ollama pull qwen2.5-coder:3b-instruct-q4_K_M`
RAM: ~2.0GB | Speed: ~4–5 tok/s on Pi 5

### Collective pool / Petals (larger models, slower, shared)

**Llama 3.1 8B Instruct**
The workhorse. A 6-person Pi cluster running Petals can host this at full precision.
Noticeably more capable than 3B models for complex reasoning, longer context, and
nuanced instructions.
License: Llama 3.1 Community License
Petals model ID: `meta-llama/Meta-Llama-3.1-8B-Instruct`

**Mistral 7B Instruct v0.3**
Alternative to Llama 3.1 8B. Apache 2.0 license (more permissive). Similar capability.
Good choice if license terms matter for your use case.
License: Apache 2.0
Petals model ID: `mistralai/Mistral-7B-Instruct-v0.3`

---

## License summary

| Model family | License | Commercial use | Distribution |
|---|---|---|---|
| Phi-3/3.5 (Microsoft) | MIT | ✅ Yes | ✅ Yes |
| Gemma 2 (Google) | Gemma | ✅ Yes (with limits) | ⚠️ Restricted |
| Mistral / Mixtral | Apache 2.0 | ✅ Yes | ✅ Yes |
| Qwen2.5 (Alibaba) | Apache 2.0 | ✅ Yes | ✅ Yes |
| Llama 3.x (Meta) | Llama Community | ✅ Yes (<700M MAU) | ⚠️ Restricted |
| DeepSeek | DeepSeek License | ⚠️ Check terms | ⚠️ Restricted |

For a private friend group, all of the above are usable. The license differences matter
primarily if you're redistributing software that includes the weights.

---

## Where to download models

**Hugging Face Hub**
The primary repository for open-weight models. Every model above is available here.
Use the GGUF format for llama.cpp and Ollama compatibility.
→ https://huggingface.co/models

**Ollama Library**
Curated list with pre-quantized models, pull commands, and memory requirements.
The easiest path — `ollama pull` handles download and format conversion.
→ https://ollama.com/library

**LM Studio model catalog**
Alternative browser-based download interface with visual VRAM/RAM requirements.
Useful for exploring before committing to a download.
→ https://lmstudio.ai/models

---

## Fine-tuning your own model on group data

Fine-tuning creates a model that knows your group's specific context, terminology,
and preferences. A 3B model fine-tuned on your group's notes, documentation, and
workflows will outperform a 70B model on tasks specific to your domain.

### What you need

- A Pi 5 (8GB) or any machine with 8GB+ RAM
- Unsloth (memory-efficient fine-tuning framework)
- A dataset of your group's text (notes, docs, Slack/Matrix exports, etc.)
- ~4–12 hours of compute time per training run

### Dataset preparation

Format your training data as instruction-response pairs:

```json
[
  {
    "instruction": "What's our deployment process for the API?",
    "response": "Our API deploys via the deploy.sh script in /scripts. Run it with --env prod for production. Always run tests first with npm test. Requires VPN to be active."
  },
  {
    "instruction": "Who owns the database migrations?",
    "response": "Database migrations are owned by the backend team. Alice is the primary contact. All migrations go in /db/migrations and must be reviewed before merging."
  }
]
```

The more specific to your actual context, the better. Generic instructions add noise.

### Training with Unsloth

```bash
pip install unsloth

python3 << 'EOF'
from unsloth import FastLanguageModel
from datasets import Dataset
import json

# Load base model
model, tokenizer = FastLanguageModel.from_pretrained(
    model_name="unsloth/Phi-3.5-mini-instruct",
    max_seq_length=2048,
    load_in_4bit=True,  # essential for Pi (reduces RAM)
)

# Load your dataset
with open("group-knowledge.json") as f:
    data = json.load(f)

dataset = Dataset.from_list(data)

# Fine-tune
trainer = FastLanguageModel.get_peft_model(model, ...)
trainer.train()

# Save
model.save_pretrained("./our-group-model")
tokenizer.save_pretrained("./our-group-model")
EOF
```

Convert to GGUF for Ollama:
```bash
cd llama.cpp
python3 convert_hf_to_gguf.py ../our-group-model --outfile ../our-group-model.gguf
./build/bin/llama-quantize ../our-group-model.gguf ../our-group-model-q4.gguf Q4_K_M

# Import into Ollama
cat > Modelfile << 'EOF'
FROM ./our-group-model-q4.gguf
SYSTEM "You are the group assistant. You know our team's context, tools, and workflows."
EOF
ollama create our-group-model -f Modelfile
```

### When fine-tuning is worth it

Fine-tuning is worth the effort when:
- Your group has a specific domain (technical field, company, project)
- You find yourselves repeatedly explaining the same context to the base model
- The base model doesn't know your terminology or conventions

Fine-tuning is not worth it when:
- You just want a better general assistant (get a bigger model instead)
- Your knowledge base has fewer than ~500 high-quality examples
- Your context changes frequently (a fine-tuned model is a snapshot)

The lightweight alternative: a detailed SOUL.md + CLAUDE.md (or their OpenClaw
equivalents) with your group's context embedded. This works surprisingly well and
requires no training time.


---


## 5.3 The exit strategy — provider independence roadmap


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


---

> **Going further with networking.** The Tailscale setup in Part 3 is sufficient for most
> groups. Part 6 covers the full depth: ACLs for per-service access control, Ansible playbooks
> for managing all nodes simultaneously, and Yggdrasil as the option for groups who don't
> want to depend on any central key infrastructure at all.

---


---

# Part 6 — Mesh networking deep dive
*Tailscale ACLs, Ansible config push, and Yggdrasil for full decentralisation*


# Mesh Network — Architecture and Config Push

## What the mesh needs to do

1. Connect all nodes privately (encrypted, no public IPs required)
2. Allow services on one node to be reachable from any other node
3. Allow config updates to be pushed to all nodes simultaneously
4. Work across different ISPs, NAT types, and home routers without port forwarding

---

## Option 1: Tailscale (recommended for most groups)

Tailscale uses WireGuard under the hood with a key management server (Tailscale's
own infrastructure). This means you trust Tailscale for key management but not for
your traffic — all data is encrypted end-to-end between your nodes.

**Free tier:** 100 devices, unlimited bandwidth, 3 users (add more with a free account
per person — each person controls their own devices).

### Network topology for a friend group

Each person's Pi joins the Tailscale network. Tailscale assigns each Pi a stable IP
(in the 100.x.x.x range) and a DNS hostname. Services (NextCloud, Navidrome, OpenClaw)
are accessed via these hostnames.

```
alice-node.tail12345.ts.net  →  100.64.0.1  (Alice's Pi, gateway host)
bob-node.tail12345.ts.net    →  100.64.0.2  (Bob's Pi)
charlie-node.tail12345.ts.net →  100.64.0.3  (Charlie's Pi)
casaos-host.tail12345.ts.net  →  100.64.0.4  (shared services host)
```

### Access controls (Tailscale ACLs)

Restrict which nodes can reach which services. Example: only nodes in the group
can reach the OpenClaw gateway; Navidrome and NextCloud are reachable from mobile
devices too.

```json
{
  "acls": [
    {
      "action": "accept",
      "src": ["group:friends"],
      "dst": ["tag:openclaw-gateway:18789"]
    },
    {
      "action": "accept",
      "src": ["group:friends"],
      "dst": ["tag:services:4533,8080,4533"]
    }
  ],
  "groups": {
    "group:friends": [
      "alice@example.com",
      "bob@example.com",
      "charlie@example.com"
    ]
  }
}
```

### Subnet routing (expose local LAN through mesh)

If one person's Pi should act as an exit node or expose local LAN services:

```bash
sudo tailscale up --advertise-routes=192.168.1.0/24 --accept-routes
# Then approve the route in Tailscale admin panel
```

---

## Option 2: Yggdrasil (fully decentralized, no central authority)

Yggdrasil is a mesh networking protocol that doesn't depend on any central
infrastructure. No Tailscale servers involved. Your nodes find each other directly.

**When to use Yggdrasil instead of Tailscale:**
- Your group doesn't want to trust any external key management
- You want the mesh to survive even if Tailscale (the company) disappears
- You're building something meant to be fully decentralized

**Tradeoff:** More configuration, less polished tooling, harder to debug.

```bash
# Install on each Pi
sudo apt install yggdrasil

# Generate config
yggdrasil -genconf | sudo tee /etc/yggdrasil/yggdrasil.conf

# Edit config to add known peers (other nodes' public keys or public Yggdrasil peers)
sudo nano /etc/yggdrasil/yggdrasil.conf

# Start
sudo systemctl enable yggdrasil
sudo systemctl start yggdrasil

# Get your IPv6 address (Yggdrasil uses IPv6)
sudo yggdrasilctl getSelf
```

Each node gets a stable globally-routable IPv6 address derived from its public key.
No IP assignment service needed — the address is cryptographically determined.

---

## Config push with Ansible

Once your group has 3+ nodes, managing them individually becomes painful. Ansible
lets you push changes to all nodes simultaneously with one command.

### Setup (on any one machine — not the Pis)

```bash
pip install ansible

# SSH key setup (do this once)
ssh-keygen -t ed25519 -f ~/.ssh/openclaw-net-key
# Copy public key to each Pi:
ssh-copy-id -i ~/.ssh/openclaw-net-key pi@alice-node.tail12345.ts.net
ssh-copy-id -i ~/.ssh/openclaw-net-key pi@bob-node.tail12345.ts.net
```

### Inventory file

```ini
# inventory.ini
[nodes]
alice-node.tail12345.ts.net
bob-node.tail12345.ts.net
charlie-node.tail12345.ts.net

[services]
casaos-host.tail12345.ts.net

[all:vars]
ansible_user=pi
ansible_ssh_private_key_file=~/.ssh/openclaw-net-key
```

### Common operations

```bash
# Push OpenClaw config update to all nodes
ansible nodes -i inventory.ini -m copy \
  -a "src=config/openclaw-config.json dest=~/.openclaw/config.json"

# Restart OpenClaw on all nodes
ansible nodes -i inventory.ini -m shell -a "openclaw restart"

# Pull new model on all nodes simultaneously
ansible nodes -i inventory.ini -m shell \
  -a "ollama pull phi3.5:3.8b-mini-instruct-q4_K_M"

# Run openclaw doctor on all nodes, collect results
ansible nodes -i inventory.ini -m shell -a "openclaw doctor 2>&1" \
  --one-line

# Update all Pis simultaneously
ansible nodes -i inventory.ini -m shell \
  -a "sudo apt update && sudo apt upgrade -y"

# Check which Ollama models are loaded on each node
ansible nodes -i inventory.ini -m shell -a "ollama list"
```

### Playbook for new member onboarding

When a new person joins the group, run one playbook that sets up their Pi completely:

```yaml
# onboard-new-member.yml
---
- name: Onboard new group member
  hosts: "{{ target_node }}"
  become: yes
  tasks:
    - name: Install core dependencies
      apt:
        name: [git, curl, build-essential, cmake, python3, python3-pip, nodejs, docker.io]
        state: present
        update_cache: yes

    - name: Install Ollama
      shell: curl -fsSL https://ollama.com/install.sh | sh

    - name: Pull assistant model
      shell: ollama pull phi3.5:3.8b-mini-instruct-q4_K_M
      become_user: pi

    - name: Pull background model
      shell: ollama pull llama3.2:1b-instruct-q8_0
      become_user: pi

    - name: Install OpenClaw
      shell: npm install -g openclaw
      become_user: pi

    - name: Install Tailscale
      shell: curl -fsSL https://tailscale.com/install.sh | sh

    - name: Copy OpenClaw base config
      copy:
        src: config/openclaw-base-config.json
        dest: /home/pi/.openclaw/config.json
        owner: pi

    - name: Run openclaw doctor
      shell: openclaw doctor
      become_user: pi
      register: doctor_output

    - name: Show doctor results
      debug:
        msg: "{{ doctor_output.stdout }}"
```

Run with:
```bash
ansible-playbook -i inventory.ini onboard-new-member.yml \
  --extra-vars "target_node=new-member-node.tail12345.ts.net"
```

---

## Hardware mesh profiles

The group can maintain a shared repository of hardware configuration profiles
that Ansible pushes to nodes. This is how you ensure every Pi in the group has
consistent settings.

```
group-config/
├── inventory.ini
├── playbooks/
│   ├── onboard.yml
│   ├── update-all.yml
│   └── push-openclaw-config.yml
├── config/
│   ├── openclaw-base-config.json
│   ├── ollama-config.json
│   └── tailscale-acls.json
└── templates/
    ├── soul-md-template.md
    └── openclaw-config.json.j2
```

Store this in a private Gitea repository. When you make a config change, commit it
and run the relevant playbook. Every node in the group stays synchronized.

This is the "mesh network profiles to push to hardware" approach — infrastructure
as code for a friend group's distributed compute cluster.


---

## 6.4 Inference stack detail — Ollama, llama.cpp, and Petals


# Inference Stack — Ollama, llama.cpp, and Petals on Raspberry Pi

## The three layers

Every node runs all three. They serve different purposes:

| Tool | Purpose | When it's used |
|---|---|---|
| **Ollama** | Primary inference runtime | All personal agent tasks |
| **llama.cpp** | Low-level engine Ollama uses; also direct CLI | Fine-grained control, benchmarking |
| **Petals** | Distributed inference across all nodes | Heavy tasks routed to the collective pool |

---

## Ollama on Pi 5

Ollama is the simplest path. Install once, pull models, point OpenClaw at it.

```bash
curl -fsSL https://ollama.com/install.sh | sh

# Start (runs as a service after install)
sudo systemctl status ollama

# Pull your models
ollama pull phi3.5:3.8b-mini-instruct-q4_K_M   # 2.4GB
ollama pull llama3.2:1b-instruct-q8_0           # 1.3GB

# Test
ollama run phi3.5:3.8b-mini-instruct-q4_K_M "What's 2+2? Answer in one word."
```

### Ollama performance tuning for Pi

```bash
# Set number of threads (Pi 5 has 4 cores, leave 1 for OS)
export OLLAMA_NUM_PARALLEL=1
export OLLAMA_MAX_LOADED_MODELS=2  # keep both models in memory
export OLLAMA_NUM_THREAD=3

# Make persistent
echo 'OLLAMA_NUM_PARALLEL=1' | sudo tee -a /etc/environment
echo 'OLLAMA_MAX_LOADED_MODELS=2' | sudo tee -a /etc/environment
echo 'OLLAMA_NUM_THREAD=3' | sudo tee -a /etc/environment
```

### Keeping models in memory

By default Ollama unloads models after 5 minutes of inactivity. For an always-on
agent, you want your primary model always loaded:

```bash
# Keep model loaded indefinitely
curl http://localhost:11434/api/generate -d '{
  "model": "phi3.5:3.8b-mini-instruct-q4_K_M",
  "keep_alive": -1
}'
```

Put this in a cron job to run at boot: `@reboot curl http://localhost:11434/api/generate -d '{"model":"phi3.5:3.8b-mini-instruct-q4_K_M","keep_alive":-1}'`

---

## llama.cpp on Pi 5

llama.cpp gives you direct control and is the engine under Ollama. Build it once;
use it for benchmarking, fine-tuning workflows, and Petals.

```bash
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp

# Build with OpenBLAS for CPU optimization
cmake -B build \
  -DLLAMA_BLAS=ON \
  -DLLAMA_BLAS_VENDOR=OpenBLAS \
  -DCMAKE_BUILD_TYPE=Release

cmake --build build --config Release -j4
# Takes ~20 minutes on Pi 5

# Test
./build/bin/llama-cli \
  -m /path/to/phi-3.5-mini-q4_K_M.gguf \
  -p "Hello, are you working?" \
  -n 50
```

### Performance benchmarks (Pi 5, 8GB, Q4_K_M models)

| Model | Tokens/sec (prompt) | Tokens/sec (generation) | Notes |
|---|---|---|---|
| Llama 3.2 1B Q8 | ~25 t/s | ~10 t/s | Fastest, background tasks |
| Phi-3.5 Mini Q4 | ~8 t/s | ~4 t/s | Best quality/speed balance |
| Llama 3.2 3B Q4 | ~6 t/s | ~3 t/s | Good general purpose |
| Mistral 7B Q4 | ~3 t/s | ~1.5 t/s | Slow but capable |

These are Pi 5 (8GB) numbers with OpenBLAS and 3 threads. Actual numbers vary based
on prompt length and RAM pressure from other running processes.

---

## Petals — distributed inference across the group

Petals splits a large model's layers across multiple nodes. Each node hosts a portion
of the model and processes inference requests in sequence, passing activations between
nodes over the network.

### How it works

```
Your prompt
    ↓
Petals client (any node)
    ↓
Node 1 (layers 1–8)
    ↓
Node 2 (layers 9–16)
    ↓
Node 3 (layers 17–24)
    ↓
Node 4 (layers 25–32)
    ↓
Response
```

Each Pi hosts 8–10 layers of a 32-layer model. A 6-person group with Pi 5s (8GB each)
can collectively run Llama 3.1 8B at full precision — a model that wouldn't fit
comfortably on any single Pi.

### Setting up Petals on each node

```bash
# On each Pi, in a dedicated virtual environment
python3 -m venv ~/petals-env
source ~/petals-env/bin/activate
pip install petals

# Start serving (each node automatically finds the others via DHT)
python3 -m petals.cli.run_server \
  meta-llama/Meta-Llama-3.1-8B-Instruct \
  --host_maddrs "/ip4/0.0.0.0/tcp/31330" \
  --num_blocks 8 \
  --device cpu
```

Each node contributes 8 blocks (layers). With 6 nodes, all 48 blocks of the 8B model
are covered. If a node goes offline, Petals routes around it (degraded performance
but doesn't crash).

### Running inference against the Petals pool

```python
from transformers import AutoTokenizer
from petals import AutoDistributedModelForCausalLM

tokenizer = AutoTokenizer.from_pretrained("meta-llama/Meta-Llama-3.1-8B-Instruct")
model = AutoDistributedModelForCausalLM.from_pretrained(
    "meta-llama/Meta-Llama-3.1-8B-Instruct"
)

inputs = tokenizer("Analyze this situation:", return_tensors="pt")
outputs = model.generate(**inputs, max_new_tokens=200)
print(tokenizer.decode(outputs[0]))
```

### OpenClaw integration with Petals

Petals exposes an OpenAI-compatible REST API via a proxy. Run this on the gateway node:

```bash
python3 -m petals.cli.run_server \
  meta-llama/Meta-Llama-3.1-8B-Instruct \
  --host_maddrs "/ip4/0.0.0.0/tcp/31330" \
  --public_api \
  --public_api_port 8080
```

Then point OpenClaw at it:

```json
{
  "ai": {
    "modelOverrides": {
      "heavy": {
        "baseURL": "http://127.0.0.1:8080/v1",
        "model": "meta-llama/Meta-Llama-3.1-8B-Instruct"
      }
    }
  }
}
```

### Petals latency reality check

Inter-node latency adds up. With nodes in different homes connected via Tailscale:

- 3 nodes on same LAN: ~50–100ms per forward pass
- 6 nodes across internet (Tailscale): ~200–500ms per forward pass
- 200 generation tokens: 40–100 seconds total

This is acceptable for async tasks. It's not acceptable for real-time chat.

**The pattern that works:**
- Personal Ollama (local model) → interactive chat, responsive
- Petals collective → long research tasks, analysis, summarization (run in background, results delivered when done)

Configure OpenClaw to route long-context or `heavy` tasks to Petals,
and interactive conversation to your local Ollama. Your agent handles the routing
automatically based on the `modelOverrides` config.

---

## Keeping models up to date

New open-weight model releases happen frequently. When a better small model comes out,
you want to update all nodes simultaneously:

```bash
# Pull new model on all nodes via Ansible
ansible nodes -i inventory.ini -m shell \
  -a "ollama pull phi4-mini:3.8b-instruct-q4_K_M"

# Update OpenClaw config to use new model (push via Ansible)
ansible nodes -i inventory.ini -m lineinfile \
  -a "path=~/.openclaw/config.json regexp='phi3.5' line='phi4-mini:3.8b-instruct-q4_K_M'"

# Restart OpenClaw everywhere
ansible nodes -i inventory.ini -m shell -a "openclaw restart"
```

Monitor r/LocalLLaMA for new model releases. When a new small model beats Phi-3.5 Mini
on reasoning benchmarks, update the group within a week.


---

> **Standing on shoulders.** The sovereign stack described in this guide didn't emerge from
> nowhere. Part 7 names the projects, people, and communities that proved it was possible —
> and provides the full technical reading list for going deeper on any layer of the stack.

---


---

# Part 7 — Inspiration and references
*The projects, people, and reading list that made this stack possible*


## 7.1 Inspiration — projects and people


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


---


## 7.2 Technical references


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


---

## Quick start checklist

Work through this in order. Don't move on until the current step is confirmed working.

### Foundation (do first)
- [ ] Every person has Pi 5 (8GB) with NVMe SSD, imaged with Raspberry Pi OS Lite 64-bit
- [ ] Booting from NVMe (not SD card)
- [ ] All core dependencies installed (Node.js 20, Docker, Python 3, cmake, git)
- [ ] Everyone on Tailscale — ping tests pass between all nodes
- [ ] Ansible installed, SSH keys distributed to all nodes, inventory.ini tested

### Inference
- [ ] Ollama installed on every node
- [ ] Two models pulled per node: primary (phi3.5 3B or similar) + fast (llama3.2 1B)
- [ ] `curl http://localhost:11434/api/tags` returns model list on every node
- [ ] llama.cpp compiled (takes ~20 min — start it, walk away)
- [ ] Petals running in background on all nodes
- [ ] Petals can load Llama 3.1 8B across the collective pool

### OpenClaw
- [ ] OpenClaw installed on each node (or one shared gateway)
- [ ] Gateway bound to Tailscale IP only — never 0.0.0.0
- [ ] Pointed at local Ollama: `"baseURL": "http://127.0.0.1:11434/v1"`
- [ ] Model overrides configured — heartbeat/subagent on 1B model
- [ ] `openclaw doctor` shows no critical warnings
- [ ] Messaging channel (Telegram / Matrix) connected and tested
- [ ] Per-user workspace isolation configured
- [ ] Action approvals enabled for destructive operations
- [ ] Skills locked to verified sources only
- [ ] Everyone has written their SOUL.md (personality + boundaries)

### Shared services
- [ ] CasaOS running on host node
- [ ] NextCloud accessible from all nodes via Tailscale
- [ ] Navidrome running, at least one person's music library synced
- [ ] Matrix homeserver running, everyone has accounts
- [ ] SearXNG running, OpenClaw web search pointed at it
- [ ] Pi-hole + Unbound running, set as DNS on all devices
- [ ] Vaultwarden running, everyone migrated from cloud password manager

### Group coordination
- [ ] Ansible playbook tested — can push config to all nodes in one command
- [ ] Everyone knows `/new` (fresh context) and `/btw` (side question)
- [ ] Gateway operator has `openclaw doctor` in their weekly routine
- [ ] Group Gitea repository holds shared config, playbooks, and SOUL.md templates


---

## Cost breakdown

### Hardware (one-time per person)

| Component | Model | Cost |
|---|---|---|
| Single-board computer | Raspberry Pi 5 (8GB RAM) | ~£80 |
| Storage | 512GB NVMe SSD + M.2 HAT+ | ~£45 |
| Cooling | Official active cooler | ~£10 |
| Power supply | 27W USB-C | ~£15 |
| Case | Any Pi 5 compatible | ~£10–20 |
| **Per person total** | | **~£160–170** |
| **Group total (6 people)** | | **~£960–1,020** |

### Ongoing costs (monthly per person)

| Item | Cost |
|---|---|
| Electricity (Pi 5, realistic workload, £0.28/kWh) | ~£3–6 |
| Tailscale (free tier, up to 100 devices) | £0 |
| API inference costs | £0 |
| SaaS subscriptions replaced | £0 |
| **Total** | **£3–6/month electricity only** |

### What you're replacing

| Service cancelled | Typical monthly cost |
|---|---|
| Spotify | £11 |
| Google One (2TB) | £10 |
| GitHub Pro | £4 |
| 1Password | £3 |
| ChatGPT Plus or Claude Pro | £18–20 |
| **Subscriptions saved per person** | **~£46–48/month** |

At £3–6/month in electricity vs £46–48 in subscriptions, the hardware pays for itself in
roughly 4 months. After that, you're ahead.


---

## Security checklist

Run through this after initial setup and after every major update.

### Gateway security
- [ ] Gateway bound to Tailscale IP (not `0.0.0.0`)
- [ ] Auth tokens generated and distributed per user (`openssl rand -hex 32`)
- [ ] `openclaw doctor` shows no critical warnings
- [ ] Skills locked to verified sources: `"allowSources": ["clawhub:verified"]`
- [ ] Group approval required for all skill installs

### Per-user safety
- [ ] Action approvals enabled for destructive operations (delete, move, shell exec)
- [ ] Each person has a SOUL.md with explicit "never do this" boundaries
- [ ] Workspace isolation configured — no cross-user memory bleed
- [ ] Rate limits set per user to prevent runaway cost

### Network
- [ ] Pi-hole running — no DNS queries going to Google or Cloudflare
- [ ] Tailscale ACLs restrict which nodes reach which services
- [ ] No services exposed on public IPs — Tailscale only
- [ ] Ollama API port (11434) not accessible outside Tailscale network

### Ongoing
- [ ] `openclaw doctor` run weekly by gateway operator
- [ ] `openclaw doctor --fix` run after every version update
- [ ] Review logs at `~/.openclaw/logs/` if any agent behaviour seems unexpected
- [ ] New skills reviewed by the group before install — one at a time, watch for 3 days

### CVE awareness
- **CVE-2026-25253** (patched v2026.1.29) — if your gateway was public before this patch,
  rotate all API keys and OAuth tokens it had access to.
- **Windows SMB credential leak** (patched v2026.3.22) — Windows gateway hosts on older
  versions should update immediately and rotate credentials.
- **ClawHavoc** (ongoing) — 1,400+ malicious skills identified on ClawHub. VirusTotal
  scanning helps but prompt-injection attacks can evade it. Verified sources only.


---

## Migration from Clawdbot / Moltbot

If anyone in your group installed OpenClaw during its viral wave in January–February 2026,
the v2026.3.22 update removed all backward compatibility for old naming conventions.
Old config is silently ignored — the agent boots but has no memory.

**Signs you need this:**
- Agent feels like it forgot everything after the update
- `openclaw doctor` reports missing config or state
- `ls -la ~/` shows `.clawdbot` or `.moltbot` directories but no `.openclaw`

**Fix (run on each affected machine):**

```bash
# Rename environment variables
sed -i 's/CLAWDBOT_/OPENCLAW_/g; s/MOLTBOT_/OPENCLAW_/g' ~/.env

# Also check shell config files
grep -l 'CLAWDBOT_\|MOLTBOT_' ~/.bashrc ~/.zshrc ~/.profile 2>/dev/null |   xargs -I{} sed -i 's/CLAWDBOT_/OPENCLAW_/g; s/MOLTBOT_/OPENCLAW_/g' {}

# Move state directory
mv ~/.moltbot ~/.openclaw 2>/dev/null || mv ~/.clawdbot ~/.openclaw 2>/dev/null

# Rename config file
mv ~/.openclaw/moltbot.json ~/.openclaw/openclaw.json 2>/dev/null
mv ~/.openclaw/clawdbot.json ~/.openclaw/openclaw.json 2>/dev/null

# Run automated migration
openclaw doctor --fix

# Migrate cron job metadata
openclaw cron purge --older-than 7d

# Restart
openclaw restart
```

After restarting, send your agent a message. It should greet you with context from your SOUL.md
and remember previous sessions. If it feels like a stranger, check `openclaw doctor` output —
the state directory migration may not have found everything.


---

*This guide was assembled from the open-claw and open-claw-net skill files. It is a living
document — update it when the group discovers new patterns, better models, or improved
configurations. The best version of this guide is the one annotated with your group's
actual experience.*

*Hardware costs approximate as of Q1 2026 in GBP. Electricity rates vary by location.*

*All software referenced is open-source. No commercial relationships. No sponsored content.*
