# 🧠 Brainstorm Skill

> Structured brainstorming sessions with persistent state. Open a topic, discuss across multiple sessions, finalize decisions into the document chain.

The `/brainstorm` skill creates a persistent file under `.atl/brain-storms/` (or `~/.claude/` for `--global` scope, or the team repo for `--team`), pins it into the scope's `CLAUDE.md` so the next session cannot miss it, and routes settled decisions through `brain-storms/ → docs/ → CLAUDE.md` when you `/brainstorm done`.

Three scopes: project (default), `--global`, `--team`. Active-brainstorm pinning shipped in `brainstorm@1.1.0`. Backlog discipline ensures every "do it later" item lands in `.atl/backlog.md` before a brainstorm closes.

## 📚 Documentation

Full docs live at **[agentteamland.github.io/docs](https://agentteamland.github.io/docs/)**.

Most relevant sections:

- [`/brainstorm` skill page](https://agentteamland.github.io/docs/skills/brainstorm) — full reference for `start` / `done` flow, three scopes, document chain, backlog discipline
- [Knowledge system](https://agentteamland.github.io/docs/guide/knowledge-system) — where brainstorm decisions land alongside journal + wiki
- [Claude Code conventions](https://agentteamland.github.io/docs/guide/claude-code-conventions) — the `<!-- brainstorm:active -->` marker block this skill writes
- [Install via `atl`](https://agentteamland.github.io/docs/cli/install) — `atl install brainstorm` (the legacy `/team install` was retired in `team-manager@2.0.0`)

## License

MIT.
