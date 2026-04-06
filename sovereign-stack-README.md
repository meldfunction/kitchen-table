# Sovereign Stack — Package Contents

## What's here

```
sovereign-stack-guide.md   ← THE GUIDE — read this
stitch_guide.py            ← Python script that built the guide from source files
source-skills/             ← Original modular skill files
  open-claw/               ← OpenClaw basics + advanced config
  open-claw-net/           ← Decentralised Pi cluster + self-hosted services
```

## How to use this

**Just want to read:** Open `sovereign-stack-guide.md` in any Markdown viewer.
It renders well in VS Code, Obsidian, GitHub, or any browser via a Markdown plugin.

**Want to update or extend:** Edit the source files in `source-skills/`, then re-run
`python3 stitch_guide.py` to regenerate the guide.

**Want to install as OpenClaw skills:** Copy the `open-claw/` and `open-claw-net/`
directories into your OpenClaw skills path:
```bash
cp -r source-skills/open-claw ~/.openclaw/skills/
cp -r source-skills/open-claw-net ~/.openclaw/skills/
openclaw skills reload
```

## Guide structure

| Part | Topic |
|---|---|
| 1 | OpenClaw basics — CLAUDE.md, permissions, sessions, subagents, hooks |
| 2 | Advanced config — CLAUDE.md patterns, hook recipes |
| 3 | Sovereign stack — Pi hardware, software setup, inference, OpenClaw on local models |
| 4 | Self-hosted services — NextCloud, Navidrome, Matrix, SearXNG, Gitea |
| 5 | Open models — which to use, licences, fine-tuning, exit roadmap |
| 6 | Mesh networking — Tailscale, Ansible config push, Yggdrasil |
| 7 | Inspiration and references |
| Appendix | Quick start checklist, cost breakdown, security checklist, migration |

