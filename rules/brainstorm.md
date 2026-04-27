# Brainstorm Rules

## Active Brainstorm Check

There are TWO redundant signals for active brainstorms — honor both:

### Signal 1 — CLAUDE.md / README marker (auto-loaded, hard to miss)

Every active brainstorm pins itself into the scope's `CLAUDE.md` (or team `README.md`) inside an `<!-- brainstorm:active:start --> ... <!-- brainstorm:active:end -->` block at start time, and removes itself at done time. Because `CLAUDE.md` is auto-loaded into every Claude session as project instructions, the active brainstorm appears at the top of context unconditionally — no scanning required.

If you see this block at the start of a session: **read the linked brainstorm file before doing any work** that touches its topic.

### Signal 2 — Directory scan (source of truth)

The marker block is a redundancy mechanism — the directory itself is authoritative. At the start of every conversation, also check for files with `status: active` frontmatter in:

- `.claude/brain-storms/` (current project)
- `~/.claude/brain-storms/` (global, cross-project)
- For team-scoped brainstorms: `~/.claude/repos/agentteamland/*/brain-storms/`

If a `status: active` file exists but no marker block does (e.g., the brainstorm was created before this rule existed, or someone hand-edited files), restore the marker via the same format the `start` skill writes — and inform the user that the marker was recovered.

### What to do when an active brainstorm is detected

1. Read the file and understand the full context
2. Inform the user that there is an active brainstorm and which one
3. Update the file on every message cycle (see "Keeping It Alive" below)

## Keeping It Alive (When an Active Brainstorm Exists)

On every message cycle:
- **When a message is received:** Read the brainstorm file (to recall context)
- **After responding:** Write new decisions, rejected ideas, and their reasons to the file

### What should be in the file:
- NOT just "Decision X was made" -> "X was proposed, rejected due to Y, a modified version of X was accepted due to Z"
- The user's exact statements (at important points) -- the spirit lives there
- Open questions and next steps
- Chronological flow -- which idea came after which

### Purpose:
The brainstorm file should be detailed and spirited enough that a Claude reading it in a new context can continue as if it had been present in that conversation.

## Document Chain

Every discussion and decision follows a three-layer chain:

```
brain-storms/ (process) -> docs/ (outcome) -> CLAUDE.md (summary)
                     \
                       backlog.md (deferred items)
```

- No decision is made without a brainstorm
- Brainstorm files are never deleted (historical record)
- If decisions change, a new brainstorm is opened and a "superseded by X" note is added to the old one

## Backlog Discipline

Every item marked as "not doing now, later" during a brainstorm must be reflected in **`.claude/backlog.md`**. This is critical for preventing scope creep -- we record what we're not doing now so we remember when a feature need arises in the future.

### When to add to the backlog:
- When a sub-topic is deemed "premature, let's defer" during a brainstorm -> write to the backlog **immediately**, don't say "later"
- When a decision item is explicitly marked as "not included in this step"
- Topics marked as Tier 3 that are more than a single sentence and require a permanent record
- Everything noted as "we'll do this later" during development

### Format:
- **Prepend** (newest on top) -- added to the beginning of the `.claude/backlog.md` file, older items stay below
- For each item: date + category heading + context link + detailed topic description + "when does this come up" note + related resources
- Follow the template at the top of the file

### Mandatory check during brainstorm done:
When `/brainstorm done` is called, scan all "deferred" notes in the brainstorm file and **ensure each one has a corresponding entry in the backlog**. If any are missing, ask the user and add them. This is a checklist step before closing the brainstorm.

### Removing from the backlog:
When an item is implemented, it is **deleted** from the backlog (not marked as done and left). Because it is now part of the active infrastructure, the relevant docs/CLAUDE.md is updated instead.
