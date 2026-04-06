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
