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
