---
name: claude
description: Use when the user asks to run a separate Claude Code CLI instance (claude exec, claude -p) or references spawning another Claude for code analysis, refactoring, or automated editing
---

# Claude Skill Guide

This skill delegates prompts to a **separate Claude Code CLI process** (`claude -p`), running it as an external tool. This is useful for parallel analysis, second opinions, or running tasks in isolation.

## Running a Task
1. Ask the user (via `AskUserQuestion`) which model to run (`opus`, `sonnet`, or `haiku` — or full model IDs like `claude-opus-4-6`, `claude-sonnet-4-6`, `claude-haiku-4-5-20251001`) AND which permission mode to use (`default`, `acceptEdits`, `plan`, or `bypassPermissions`) AND which effort level to use (`low`, `medium`, `high`, or `max`) in a **single prompt with three questions**.
2. Select the permission mode required for the task; default to `--permission-mode plan` for read-only analysis, `--permission-mode acceptEdits` for edit tasks.
3. Assemble the command with the appropriate options:
   - `--model <MODEL>` (alias or full model ID)
   - `--permission-mode <default|acceptEdits|bypassPermissions|plan>`
   - `--effort <low|medium|high|max>`
   - `--print` / `-p` (required: non-interactive mode, print response and exit)
   - `--add-dir <DIR>` (additional directories)
   - `--allowedTools <tools...>` (restrict available tools)
   - `--disallowedTools <tools...>` (deny specific tools)
   - `--max-budget-usd <amount>` (spending limit)
   - `"your prompt here"` (as final positional argument)
4. **IMPORTANT**: Always use `-p` / `--print` flag to run in non-interactive mode.
5. When continuing a previous session, use `claude --continue` or `claude -c` to continue the most recent conversation. Use `claude --resume <session-id>` or `claude -r <session-id>` to resume a specific session.
6. Run the command, capture stdout/stderr, and summarize the outcome for the user.
7. **After Claude completes**, inform the user: "You can resume this Claude session at any time by saying 'claude resume' or 'claude continue', or asking me to continue with additional analysis or changes."

### Quick Reference
| Use case | Permission mode | Key flags |
| --- | --- | --- |
| Read-only review or analysis | `plan` | `-p --permission-mode plan --model <MODEL> "prompt"` |
| Apply local edits (auto-approve edits) | `acceptEdits` | `-p --permission-mode acceptEdits --model <MODEL> "prompt"` |
| Full auto (bypass all permissions) | `bypassPermissions` | `-p --permission-mode bypassPermissions --model <MODEL> "prompt"` |
| Continue most recent session | Inherited | `claude -c -p "follow-up prompt"` |
| Resume specific session | Inherited | `claude -r <session-id> -p "follow-up prompt"` |
| Run in separate worktree | Match task | `-p --worktree --model <MODEL> "prompt"` |
| Budget-limited run | Match task | `-p --max-budget-usd 5.00 --model <MODEL> "prompt"` |

## Following Up
- After every `claude` command, immediately use `AskUserQuestion` to confirm next steps, collect clarifications, or decide whether to continue/resume.
- When continuing, use `claude -c -p "new prompt"` to continue the most recent conversation.
- Restate the chosen model, permission mode, and effort level when proposing follow-up actions.

## Critical Evaluation of Claude Output

The spawned Claude instance is a separate process with its own context. Treat it as a **colleague, not an authority**.

### Guidelines
- **Compare outputs** when you and the spawned Claude disagree. Your context may differ from the spawned instance's context.
- **Research disagreements** using WebSearch or documentation before accepting the spawned Claude's claims.
- **Context differences** - The spawned Claude starts fresh and may lack conversation context you already have. Provide sufficient context in the prompt.
- **Don't defer blindly** - The spawned instance can make mistakes. Evaluate its suggestions critically, especially regarding:
  - Project-specific conventions it may not know
  - Context from the current conversation it doesn't have
  - Files or changes it hasn't seen

### When the Spawned Claude is Wrong
1. State your disagreement clearly to the user
2. Provide evidence (your own analysis, web search, docs)
3. Optionally continue the session to discuss the disagreement:
   ```bash
   claude -c -p "I'm the parent Claude session following up. I disagree with [X] because [evidence]. Please reconsider."
   ```
4. Frame disagreements as discussions - the spawned instance may have a valid perspective you missed
5. Let the user decide how to proceed if there's genuine ambiguity

## Error Handling
- Stop and report failures whenever `claude --version` or a `claude -p` command exits non-zero; request direction before retrying.
- Before you use high-impact flags (`--permission-mode bypassPermissions`, `--dangerously-skip-permissions`) ask the user for permission using AskUserQuestion unless it was already given.
- When output includes warnings or partial results, summarize them and ask how to adjust using `AskUserQuestion`.
- Set a reasonable `--max-budget-usd` for long-running tasks to prevent unexpected costs.
