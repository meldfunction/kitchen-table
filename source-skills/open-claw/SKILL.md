---
name: open-claw
description: >
  Instruction guide for setting up and configuring Claude Code ("Claw") for maximum effectiveness.
  Use this skill whenever a user wants to set up Claude Code, configure their environment, write a
  CLAUDE.md file, set up hooks, configure permissions, use subagents, manage sessions, or get more
  out of Claude Code generally. Trigger this even if the user just says "help me set up claw",
  "how do I use Claude Code better", "configure my claude", or anything that suggests they want to
  optimize their Claude Code workflow — even if they don't use the word "skill" or "setup".
---

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
