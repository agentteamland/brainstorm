# 🧠 Brainstorm Skill

A Claude Code skill for structured brainstorming sessions with persistent state. Start a brainstorm, discuss ideas across multiple sessions, and finalize decisions into documentation.

## Installation

```bash
/team install https://github.com/mkurak/agent-workshop-brainstorm-skill.git
```

> Requires [Agent Team Manager](https://github.com/mkurak/agent-workshop-agent-team-manager-skill) to be installed first.

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
- Captures discussion, decisions, rejected alternatives, and reasoning
- Updates the file after every message exchange
- Survives context switches — new sessions pick up where you left off

### Finishing a Brainstorm
- Marks brainstorm as completed
- Generates/updates documentation in `.claude/docs/`
- Updates `CLAUDE.md` with a summary
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
- **Global support** — `--global` flag for cross-project topics
- **Decision tracking** — not just outcomes, but *why* decisions were made
- **Backlog discipline** — every "do it later" item is captured
- **Multiple active brainstorms** — each in its own file

## License

MIT
