# 🧠 Brainstorm Skill

A Claude Code skill for structured brainstorming sessions with persistent state. Start a brainstorm, discuss ideas across multiple sessions, and finalize decisions into documentation.

## Installation

```bash
/team install https://github.com/agentteamland/brainstorm.git
```

> Requires [Agent Team Manager](https://github.com/agentteamland/team-manager) to be installed first.

## Usage

```bash
# Start a new brainstorm
/brainstorm start Let's discuss the authentication architecture

# Start a global brainstorm (cross-project, stored in ~/.claude/)
/brainstorm start --global Agent team organization strategy

# Finalize and generate documentation
/brainstorm done
```

## How It Works

### Starting a Brainstorm
- Creates a `.claude/brain-storms/{topic}.md` file (or `~/.claude/` for global)
- **Pins the active brainstorm to the scope's `CLAUDE.md`** (or team `README.md`) inside a `<!-- brainstorm:active:start --> ... <!-- brainstorm:active:end -->` marker block — so future Claude sessions see it as part of auto-loaded project instructions and cannot miss it
- Captures discussion, decisions, rejected alternatives, and reasoning
- Updates the file after every message exchange
- Survives context switches — new sessions pick up where you left off

### Finishing a Brainstorm
- Marks brainstorm as completed
- Generates/updates documentation in `.claude/docs/`
- Updates `CLAUDE.md` with a summary
- **Removes the active-brainstorm marker** for this topic; if it was the last active one, removes the whole marker block
- Verifies all deferred items are captured in `.claude/backlog.md`

### Document Chain
```
brain-storms/ (process) → docs/ (decisions) → CLAUDE.md (summary)
                       ↘
                         backlog.md (deferred items)
```

## What's Included

| Type | File | Purpose |
|------|------|---------|
| Skill | `skills/brainstorm/skill.md` | The `/brainstorm` command |
| Rule | `rules/brainstorm.md` | Auto-loaded rules for brainstorm workflow |

## Key Features

- **Persistent state** — brainstorm files survive context resets
- **Active-brainstorm pinning** — every active brainstorm is auto-pinned to `CLAUDE.md` (or team `README.md`) so it's loaded into every Claude session as project instructions; impossible to miss
- **Global support** — `--global` flag for cross-project topics
- **Decision tracking** — not just outcomes, but *why* decisions were made
- **Backlog discipline** — every "do it later" item is captured
- **Multiple active brainstorms** — each in its own file

## License

MIT
