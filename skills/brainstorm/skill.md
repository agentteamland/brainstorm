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
participants: Mesut, Claude
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

### 5. Respond
Inform the user that the brainstorm has started, including the filename and scope, then dive into the topic.

### 6. On Subsequent Messages
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

### 5. Git Push for Team Brainstorms
After completing a brainstorm in team scope:
```bash
cd ~/.claude/repos/agentteamland/{team-name}
git add -A
git commit -m "brainstorm: {topic summary}"
git push
```

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
8. **Automatic git push for team brainstorms.** Commit + push is performed after done.
