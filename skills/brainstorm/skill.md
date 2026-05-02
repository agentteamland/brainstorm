---
name: brainstorm
description: "Start and complete brainstorming sessions. start = begin a new brainstorm, done = complete the active brainstorm and propagate to the document chain. 3 scopes: project (default), --global, --team."
argument-hint: "<start|done> [--global|--team] [initial message]"
---

# /brainstorm Skill

## Parameter Parsing

The first word determines the mode: `start` or `done`. The remaining text (if any) is the user's initial message.

## Three Scopes

| Flag | Target Directory | When |
|------|------------|----------|
| *(none)* | `.claude/brain-storms/` | Project-specific topics (default) |
| `--global` | `~/.claude/brain-storms/` | Cross-project, personal topics |
| `--team` | `~/.claude/repos/agentteamland/{team}/brain-storms/` | Topics related to the team repo (agent rules, team strategy) |

**`--team` active team detection:**
- The `readlink` result of symlinks under `~/.claude/agents/` is used to extract `~/.claude/repos/agentteamland/{team-name}/`
- If there's a single team, it's used automatically
- If there are multiple teams, the user is asked via AskUserQuestion
- If no team is found, error: "No installed team found. Run /team install first."

---

## `start` Mode

When the user says `/brainstorm start ...`:

### 1. Understand the Topic
Extract the topic from the user's message. The user does not provide a topic title -- you infer it from the message and determine an appropriate `kebab-case` filename.

### 2. Determine Scope
- If `--global` is present -> `base_dir = ~/.claude/`
- If `--team` is present -> `base_dir = ~/.claude/repos/agentteamland/{team-name}/` (detect active team)
- If neither -> `base_dir = .claude/` (project root)

### 3. Create Directory (if it doesn't exist)
Create the `{base_dir}/brain-storms/` directory if it doesn't exist.

### 4. Create File
Create the `{base_dir}/brain-storms/{name}.md` file:

```markdown
---
status: active
scope: {project|global|team}
team: {team-name or null}
date: {today's date}
participants: <user-name>, Claude
---

# {Topic Title}

## Context

{Context and motivation understood from the user's initial message}

## Discussion

### {Date} — Start

{Summary of the initial message and any first ideas}

## Open Items

- {Questions or points to discuss extracted from the initial message}
```

### 5. Pin to CLAUDE.md / README (active-brainstorm marker)

Inject a marker block into the scope's `CLAUDE.md` (or team `README.md`) so that future Claude sessions cannot miss the active brainstorm. The block is auto-loaded with the project context, making the active brainstorm impossible to overlook even when the rule's "check the directory" step is skipped.

**Target file by scope:**
- **project** → `CLAUDE.md` at the project root
- **global** → `~/.claude/CLAUDE.md`
- **team** → `~/.claude/repos/agentteamland/{team}/README.md`

**Marker block format** (HTML comments are delimiters — used to find/update/remove the block):

```markdown
<!-- brainstorm:active:start -->
## ⚠️ Active brainstorms

These topics have an in-progress brainstorm — read the file before making any decision on them.

- **[{brainstorm-name}]({relative-path-to-brainstorm-file})** ({scope}, {date}) — {one-line topic summary}
<!-- brainstorm:active:end -->
```

**Insertion rules:**
1. **If the marker block does NOT exist:** insert it near the top of the file, right after the H1 + opening description (before the first H2 heading). For project `CLAUDE.md` this typically means after the intro paragraph; for `~/.claude/CLAUDE.md` it goes right after the title; for team `README.md` it goes right after the badges/intro.
2. **If the marker block EXISTS:** add a new bullet to the list (preserve existing bullets — multiple active brainstorms can coexist). Do not duplicate a bullet for the same brainstorm.
3. **Relative path:** Use a path relative to the file you're editing (e.g., `.claude/brain-storms/foo.md` from project `CLAUDE.md`; `brain-storms/foo.md` from `~/.claude/CLAUDE.md`; `brain-storms/foo.md` from team `README.md`).
4. **One-line summary:** Distill from the brainstorm's H1 title or context — keep under ~80 chars.

### 6. Respond
Inform the user that the brainstorm has started, including the filename, scope, and that it has been pinned to the appropriate `CLAUDE.md` / `README.md`. Then dive into the topic.

### 7. On Subsequent Messages
Update the brainstorm file on every message cycle:
- Add new ideas, decisions, rejected alternatives and their reasons
- Preserve the user's important statements verbatim (in quotes)
- Maintain chronological order -- add new subheadings to the discussion section
- Update the open items list (resolved ones are removed, new ones are added)

**IMPORTANT:** The file must be detailed enough that a Claude reading it in a new context can continue as if it had been present in the conversation.

---

## `done` Mode

When the user says `/brainstorm done`:

### 1. Find Active Brainstorm
Search **in all three locations**:
- `.claude/brain-storms/` (project)
- `~/.claude/brain-storms/` (global)
- `~/.claude/repos/agentteamland/*/brain-storms/` (all teams)

Find files with `status: active`. If there are multiple, list them for the user (showing which scope each is in) and ask which one to complete.

### 2. Complete the Brainstorm File
- Change `status: active` to `status: completed`
- Add final notes at the end of the discussion section
- Update the "Open Items" section (unresolved ones remain)
- Add a "Final Decisions" section -- a summary of all definitive decisions from the entire discussion

### 3. Create/Update Docs File
Determine the docs file location based on the brainstorm's scope:
- **Project brainstorm** -> write under `.claude/docs/`
- **Global brainstorm** -> write under `~/.claude/docs/`
- **Team brainstorm** -> write under `~/.claude/repos/agentteamland/{team}/docs/`

Reference the brainstorm at the top of the file.

### 4. Update CLAUDE.md / README
- **Project brainstorm** -> update `CLAUDE.md` at the project root
- **Global brainstorm** -> update `~/.claude/CLAUDE.md`
- **Team brainstorm** -> update `~/.claude/repos/agentteamland/{team}/README.md`

Two updates happen here:
1. **Add the completed-brainstorm summary** to the appropriate section (existing behavior).
2. **Remove the active-brainstorm marker** for THIS brainstorm:
   - Find the `<!-- brainstorm:active:start -->` ... `<!-- brainstorm:active:end -->` block.
   - Remove the bullet line whose link points to this brainstorm's file.
   - **If the bullet list becomes empty after removal**, remove the entire marker block (including the H2 heading and intro line) so the file does not retain a stale "Active brainstorms" section.
   - **If other bullets remain**, keep the block intact — other brainstorms are still active.

### 5. Persisting Team Brainstorms
After completing a brainstorm in team scope, the brainstorm file lives under the team's local clone at `~/.claude/repos/agentteamland/{team-name}/.claude/brain-storms/`. The `done` flow writes the file there but does NOT push directly to `origin/main` — every public agentteamland repo is branch-protected.

To get the brainstorm into the team's main branch, open a PR (manually or via `/create-pr`):

```bash
cd ~/.claude/repos/agentteamland/{team-name}
git checkout -b brainstorm/{topic}
git add .claude/brain-storms/{topic}.md
git commit -m "brainstorm: {topic summary}"
git push -u origin brainstorm/{topic}
gh pr create --fill
```

The `/create-pr` skill in `core` automates this if installed.

### 6. Respond
Inform the user that the brainstorm is complete and list the created/updated files.

---

## Important Rules

1. **Multiple active brainstorms can exist.** Each lives independently in its own file. They can be active simultaneously across different scopes.
2. **Resilience to context breaks.** The brainstorm file is persistent state. In a new context, the rule detects active brainstorms and continues by reading the file.
3. **Filename is not requested from the user.** You infer it from the message and assign an appropriate kebab-case name.
4. **Brainstorm files are never deleted.** They remain as historical records.
5. **Each brainstorm focuses on a single topic.** Different topics go in different files.
6. **Active brainstorm search covers all three locations.** In `done` mode, project + global + all team directories are scanned.
7. **Scope is specified in frontmatter.** `scope: project|global|team`, `team: {name}` -- determines the correct target in done mode.
8. **Team brainstorms ship via PR, not direct push.** Every public `agentteamland/{team}` repo is branch-protected; the `done` flow writes the brainstorm file to the team's local clone and instructs the user to open a PR (manually or via `/create-pr`).

## Accumulated Learnings

(Auto-rebuilt by /save-learnings from learnings/*.md frontmatter. Do not edit by hand. Currently empty — populates as the skill is used and edge-case learnings accumulate.)
