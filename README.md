# skill-claude

Enable any coding agent to spawn a separate [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) instance for parallel analysis, second opinions, and automated editing.

## Prerequisites

- Claude Code CLI installed and on `PATH`
- Valid credentials configured
- Verify with `claude --version`

## Installation

### Via [skills CLI](https://github.com/vercel-labs/skills) (Recommended)

```bash
npx skills add grgong/skill-claude
```

### Manual

Copy the `skill-claude` directory into your agent's skills folder:

```bash
# Claude Code
cp -r skill-claude ~/.claude/skills/claude

# Codex / universal
cp -r skill-claude ~/.agents/skills/claude
```

## Usage

Ask your agent to delegate a task to a separate Claude instance:

```
Use a separate claude instance to review my code changes.
```

The skill will guide the agent through model, permission mode, and effort level selection, then run `claude -p` in non-interactive mode.

See [SKILL.md](SKILL.md) for full operational instructions.

## License

MIT
