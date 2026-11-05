# XBuilder for CLI

XBuilder is a dedicated IDE for the XGo language, used to build games, animations, robotics applications, and more. Code agents such as Claude Code, Codex, and dsh can read and write code, but they cannot reach XBuilder's project management, asset handling, preview, or publishing features. XBuilder CLI exposes these capabilities on the command line so agents and humans can work on the same project.

## Goals

- **Capability parity.** Anything essential you can do in the IDE, you can do from the CLI.
- **Agent-friendly.** Predictable behavior, parsable output, actionable errors, no interactive prompts.
- **Shared core.** The CLI and the IDE use the same project model, checks, and build pipeline.
- **Seamless handoff.** A project edited through the CLI opens in the IDE, and vice versa.

## Non-Goals

- No graphical editing in the CLI (sprite editing, stage layout, etc.).
- Does not replace the `xgo` command; it orchestrates at the XBuilder project level.
- No agent-specific protocol: any agent that can run commands and read output can use it.

## Commands (Draft)

| Command | Purpose |
|---|---|
| `xbuilder new` | Create a project from a template |
| `xbuilder info` | Summarize project structure: characters, scenes, assets, dependencies |
| `xbuilder check` | Check code and assets; report problems with locations |
| `xbuilder run` | Run the project, windowed or headless |
| `xbuilder build` | Build and package |
| `xbuilder asset` | Import, list, validate, rename assets |
| `xbuilder export` | Export in a distributable form |
| `xbuilder publish` | Publish to a target platform |

A typical agent loop:

```
xbuilder info --json
xbuilder check --json
xbuilder run --headless --duration 5s --json
```

## Design Principles for Agents

- Every command supports `--json` with stable, structured output.
- Distinct exit codes for success, failed checks, usage errors, and internal errors.
- Never block on input; missing arguments produce an error explaining how to fix the call.
- Errors include file, location, and a suggested fix.
- Idempotent operations; file-modifying commands support `--dry-run`.
- Headless runs return logs, events, and screenshots so agents can observe results.
- Ship a short `AGENTS.md` / `CLAUDE.md` snippet describing conventions and common commands.

## Milestones

1. **Read-only:** `info`, `check`.
2. **Run and build:** `run` (including headless), `build`.
3. **Assets and creation:** `new`, `asset`.
4. **Distribution:** `export`, `publish`, CI guidance, agent onboarding docs.

## Success Criteria

- A new agent, given only the docs, can create, modify, check, run, and fix a project without human help.
- A project moves between the CLI and the IDE without losing state.
- Agents can use CLI output directly for their next decision, with no extra parsing glue.
