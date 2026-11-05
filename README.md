# XBuilder for CLI

XBuilder is a dedicated IDE for the XGo language, used to build games, animations, robotics applications, and more. Code agents such as Claude Code, Codex, and dsh can read and write code, but they cannot reach XBuilder's project management, asset handling, or publishing features. `xb`, the XBuilder CLI, exposes these capabilities on the command line, with a GitHub-style workflow for getting projects from and to the cloud, so agents and humans can work on the same project.

## Goals

- **Capability parity.** Anything essential you can do in the IDE, you can do from `xb`.
- **Familiar workflow.** `xb clone`, `xb new`, `xb push`, and `xb pull` work like their GitHub counterparts, minus the version-control ceremony.
- **Agent-friendly.** Predictable behavior, parsable output, actionable errors, no interactive prompts.
- **Shared core.** `xb` and the IDE use the same project model, checks, and build pipeline.
- **Seamless handoff.** A project edited through `xb` opens in the IDE, and vice versa.

## Non-Goals

- No graphical editing in the CLI (sprite editing, stage layout, etc.).
- Does not replace the `xgo` command; `xb run` simply delegates to it.
- Not a general-purpose Git client. Cloud sync is built in and requires no Git knowledge.
- No agent-specific protocol: any agent that can run commands and read output can use it.

## Commands (Draft)

### Cloud workflow

| Command | Purpose |
|---|---|
| `xb clone <ProjectName>` | Download a project from the cloud into a local directory |
| `xb new <ProjectName>` | Create a new project (locally and in the cloud) from a template |
| `xb push` | Upload the current project to the cloud |
| `xb pull` | Fetch updates made elsewhere (the IDE, other machines, other agents) and apply them locally |

`xb push` needs no `add`, no `commit`, and no `.gitignore`. It uploads the project as it currently is and automatically skips content that is not part of the project's source, such as build outputs, caches, and temporary files. Use `--dry-run` to preview exactly what would be uploaded.

Because the cloud copy may be changed from other clients at any time, `xb push` stops with a clear message if the cloud version is newer, instead of overwriting it. Run `xb pull` first. If a pull touches files that were also modified locally, `xb pull` reports the affected files and never overwrites local work silently.

### Authentication

Modeled on the GitHub CLI (`gh auth`):

| Command | Purpose |
|---|---|
| `xb auth login` | Sign in through the browser (device-code flow) |
| `xb auth login --with-token` | Sign in non-interactively by reading a token from stdin |
| `xb auth status` | Show the current account and token validity |
| `xb auth logout` | Remove stored credentials |

Credentials are kept in the system keychain where available. For agents and CI, the `XB_TOKEN` environment variable takes precedence over stored credentials, so no login step is needed.

### Project workflow

| Command | Purpose |
|---|---|
| `xb info` | Summarize project structure: characters, scenes, assets, dependencies |
| `xb check` | Check code and assets; report problems with locations |
| `xb run` | Run the project; equivalent to `xgo run .` |
| `xb asset` | Import, list, validate, rename assets |
| `xb export` | Export in a distributable form |

A typical agent session:

```
xb clone MyGame
cd MyGame
xb info --json
xb check --json
xb run
xb pull
xb push
```

## Design Principles for Agents

- Commands that report structured information (`info`, `check`, `asset`, and so on) support `--json` with stable output.
- Distinct exit codes for success, failed checks, usage errors, auth errors, sync conflicts, and internal errors.
- Never block on input; missing arguments or missing login produce an error explaining how to fix the call.
- Errors include file, location, and a suggested fix.
- Idempotent operations; commands that modify files or the cloud support `--dry-run`.
- Ship a short `AGENTS.md` / `CLAUDE.md` snippet describing conventions and common commands.

## Success Criteria

- A new agent, given only the docs, can clone or create a project, modify it, check, run, fix, and sync with the cloud without human help.
- A project moves between `xb` and the IDE without losing state.
- Agents can use `xb` output directly for their next decision, with no extra parsing glue.

