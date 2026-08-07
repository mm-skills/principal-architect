---
name: memory-template
description: Canonical formats for the three-file memory system (MEMORY.md, PROJECT_INDEX.md, MODULE_INDEX.md). Read this when creating or migrating the memory directory.
---

# Memory System — Templates & Rules

The PA's memory lives in `.agents/skills/principal-architect-workspace/memory/` and consists of three files, each serving a distinct lookup dimension.

## Directory Structure

```
.agents/skills/principal-architect-workspace/
├── memory/                    ← meta layer (managing work)
│   ├── MEMORY.md              ← boot registry (always read)
│   ├── PROJECT_INDEX.md       ← full project ledger + session IDs
│   └── MODULE_INDEX.md        ← domain cross-reference
├── pr-[project-name]/         ← the work itself
└── ...
```

## Migrate-on-First-Touch

Existing repos may have `MEMORY.md` at the workspace root. On boot:
1. Check for `memory/MEMORY.md` — if found, use it.
2. If not found, check for `MEMORY.md` at workspace root.
3. If found at root: create `memory/` directory, move `MEMORY.md` → `memory/MEMORY.md`, create empty `PROJECT_INDEX.md` and `MODULE_INDEX.md` from templates below, commit.
4. If neither exists: create `memory/` with all three files from templates below.

## File 1: `memory/MEMORY.md` — Boot Registry

Read at every session start. Must stay lean regardless of project count.

```markdown
# Principal Architect Memory
<!-- Last updated: [ISO timestamp] -->

## Active Programs
| Program | Plan | Status |
|---|---|---|

## Active Projects
| Project | Status | Modules | Context |
|---|---|---|---|

## Recently Completed (last 5)
| Project | Completed | Modules | Key Outcome |
|---|---|---|---|

## Older Projects
[N] completed projects.
Full project history: [PROJECT_INDEX.md](PROJECT_INDEX.md)
Cross-cutting module index: [MODULE_INDEX.md](MODULE_INDEX.md)
```

## File 2: `memory/PROJECT_INDEX.md` — Full Project Ledger

Read for project-specific investigation or session tracing. Contains every completed project with session IDs, commit hashes, and distillation links.

```markdown
# Project Index
<!-- Full chronological record of all completed projects.
     Read when tracing session history or investigating a specific project. -->

| Project | Completed | Scope | Session | Context |
|---|---|---|---|---|
```

## File 3: `memory/MODULE_INDEX.md` — Domain Cross-Reference

Read for cross-cutting queries ("what work has touched OTA?"). Organised by architectural domains that the PA identifies based on the codebase.

```markdown
# Module Index
<!-- Cross-cutting reference. Read when you need to find what
     work has touched a specific domain or module.
     Domains are architect-defined based on the codebase architecture. -->

## [Domain Category]
| Project | Scope | Context |
|---|---|---|
```

## Maintenance Touchpoints

### Step 1 — Project Registration
- Add row to `memory/MEMORY.md` Active Projects table.
- Identify initial modules from planning research.

### Step 5 — WO Completion
- Update Active row status (e.g., "WO3/6" → "WO4/6").
- Append new modules if the WO touched a previously unrecorded area.

### Step 6 — Project Wrap-Up
1. Move project from Active → Recently Completed in `memory/MEMORY.md` (add date and key outcome).
2. If Recently Completed exceeds 5 entries, graduate the oldest to `memory/PROJECT_INDEX.md` (with session ID, commit hash, scope, and context link).
3. Add the completed project to every relevant domain category in `memory/MODULE_INDEX.md`.
4. Update the "Older Projects" count.
5. Commit all memory files.

## Key Principles

- **"Module" = architectural units meaningful to your codebase.** These might be directories, packages, layers, or named subsystems. The right granularity is whatever a future PA would search for when asking "what work has touched this area?"
- **Session IDs belong in PROJECT_INDEX.md** — they are project-level metadata, not module-level. They enable future agents on the same machine to interrogate the original session's brain record.
- **All memory files must be committed to git.** Anyone cloning the repo inherits the full development history and architectural thinking.
- **MEMORY.md must stay lean.** If it exceeds ~50 lines, something has gone wrong — archive more aggressively.
