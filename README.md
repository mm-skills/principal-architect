# 🏛️ Principal Architect

> **The Elephant remembers. The Goldfish executes.**
>
> A skill that decomposes massive coding tasks into isolated, delegatable Work Orders — keeping the architect's context sharp and the workers' context clean.

[![Harnesses](https://img.shields.io/badge/Harnesses-Antigravity_|_Claude_Code_|_Generic-blue)](#harness-support)
[![Tiers](https://img.shields.io/badge/Orchestration_Tiers-3-green)](#the-three-tier-model)
[![Pattern](https://img.shields.io/badge/Pattern-Elephant_%26_Goldfish-orange)](#the-elephant--goldfish-pattern)

---

## Why This Exists

AI coding agents are powerful — but they degrade predictably. Fill an agent's context with the entire history of a complex migration and it starts hallucinating, losing track of files, and making contradictory decisions. Ask it to "just do the next bit" and it has no memory of what came before.

This is the problem the **Elephant & Goldfish** pattern solves, and this skill is a production implementation of it.

### The Elephant & Goldfish Pattern

The pattern, widely discussed in the AI engineering community, splits agentic work into two complementary session types:

| | 🐘 The Elephant | 🐟 The Goldfish |
|---|---|---|
| **Role** | Long-term orchestrator | Transient task executor |
| **Memory** | Carries full project state across sessions | Starts fresh — zero prior context |
| **Focus** | Architecture, sequencing, conflict analysis | A single, well-bounded work order |
| **Token budget** | Lean — plans, never codes | Spends tokens on implementation |
| **In this skill** | The **Principal Architect** | The **Worker sub-agent** |

The insight is simple: *bigger context windows don't fix context pollution*. The Elephant retains what matters (project memory, architectural decisions, the dependency graph). The Goldfish gets exactly what it needs (a scoped WORKORDER.md) and nothing else — keeping it fast, focused, and cheap.

---

## What It Does

When you invoke the Principal Architect, it:

1. **Pages in memory** — reads `MEMORY.md` and the active `PROJECT.md` to restore full project state (the Elephant never forgets)
2. **Decomposes the task** — breaks it into atomic Work Orders with explicit boundary boxes, dependency ordering, and conflict analysis
3. **Creates stacked branches** — each Work Order gets its own branch, stacked linearly for clean PR submission
4. **Delegates to workers** — spawns focused sub-agents with hyper-scoped WORKORDER.md prompts (the Goldfish gets only what it needs)
5. **Re-ingests results** — reads each worker's Completion Report, updates memory, and advances to the next Work Order
6. **Submits the stack** — pushes the completed branch stack to GitHub as linked PRs

The architect **never writes code**. Workers **never see the big picture**. Each stays in its lane.

---

## The Three-Tier Model

For tasks that exceed the scope of a single Pull Request, the skill scales to a three-tier hierarchy:

```
┌──────────────────────────────────────────┐
│  Tier 1 · Chief Architect                │
│  Manages the programme-level dependency  │
│  graph across multiple PRs               │
├──────────────────────────────────────────┤
│  Tier 2 · Principal Architect            │
│  Decomposes a single PR into atomic      │
│  Work Orders and manages the branch stack│
├──────────────────────────────────────────┤
│  Tier 3 · Worker Agent                   │
│  Executes a single Work Order on the     │
│  filesystem — scoped, focused, disposable│
└──────────────────────────────────────────┘
```

> [!TIP]
> Most tasks only need Tiers 2–3. The Chief Architect (Tier 1) activates automatically when the skill detects a request that spans multiple Pull Requests.

---

## Key Features

### 🧠 Persistent Memory System
- `MEMORY.md` — cross-session index of all projects, modules, and status
- `PROJECT.md` — per-project ledger with Work Order status, session logs, and architectural decisions
- Automatic memory graduation from Active → Recently Completed → Archive

### 🔀 Stacked Branch Workflow
- Linear WO branch chain (`project/wo1-topic` → `project/wo2-topic` → ...)
- No standalone feature branch — the tip of the stack *is* the PR head
- Built-in conflict detection with serialize-vs-parallelize advisory
- Stack submission via `gt stack submit` or `gh stack`

### 📋 Structured Delegation
- WORKORDER.md templates with YAML frontmatter (`complexity: mechanical | standard | architect | interactive`)
- Boundary boxes defining exactly which files/modules the worker may touch
- Completion Report templates the worker fills in on finish
- Issue-raising authority reserved to PA/Chief tiers — workers surface findings, never file tickets

### 🐛 Debug Handoff Protocol
- Detects when a session drifts into tactical debugging
- Packages full reproduction context into a debug WORKORDER
- Hands off to an interactive worker session, preserving the architect's strategic context

### 🔌 Harness-Agnostic
- Auto-detects the hosting environment by inspecting its own toolset
- Adapters for **Antigravity** (`invoke_subagent`), **Claude Code** (`Agent`), and a **Generic** fallback
- Worker complexity tiers map to harness-specific capability levers

---

## Repository Structure

```
principal-architect/
├── SKILL.md                              # Core skill instructions
├── LICENSE
├── assets/
│   └── rules/
│       └── 00-operating-model.md         # Worker-facing governance rule
└── references/
    ├── CHIEF_ARCHITECT.md                # Tier 1 programme orchestration
    ├── STACKING_REFERENCE.md             # Branch naming, stacking, sync, merge
    ├── operating-model.md                # Delegated-work authorization model
    ├── workorder-template.md             # Canonical WORKORDER structure
    ├── memory-template.md                # MEMORY.md format & migration
    ├── debug-handoff.md                  # Interactive testing protocol
    ├── distillation-guide.md             # Session log & context distillation
    └── harnesses/
        ├── antigravity.md                # Google Antigravity adapter
        ├── claude-code.md                # Anthropic Claude Code adapter
        └── generic.md                    # Generic / other harnesses
```

---

## Installation

### Via Skill Manager (Recommended)

```bash
scripts/add.sh '{"name": "principal-architect"}'
```

### As a Git Submodule

```bash
git submodule add https://github.com/mm-skills/principal-architect .agents/skills/principal-architect
```

---

## Usage

Once installed, the skill activates when you:
- Ask for a "principal architect" or "chief architect"
- Request decomposition of a large task into phases
- Need to manage a stacked branch workflow
- Want to orchestrate a multi-PR migration

**Example prompts:**
- *"Principal architect: migrate our auth system from JWT to session tokens"*
- *"Decompose this epic into work orders and start executing"*
- *"I need a chief architect to manage a 6-PR platform migration"*

---

## Requirements

- `git` — branch management and stacking
- `gh` — GitHub CLI for PR submission
- A stacking tool (`gt` from Graphite, or `gh stack`) — for stack submission
- Write access to the workspace filesystem

---

## The Pattern in Practice

A typical session flow, mapped to the Elephant & Goldfish roles:

```
🐘 PA wakes up
│  └─ Reads MEMORY.md → pages in project state
│  └─ Reads PROJECT.md → resumes from WO3/6
│
🐘 PA plans WO4
│  └─ Creates branch: project/wo4-api-routes
│  └─ Writes WORKORDER.md with boundary box
│  └─ Checks for file overlap with WO3 (none — safe to proceed)
│
🐟 Worker spawned for WO4
│  └─ Reads only WORKORDER.md — no project history
│  └─ Implements the change
│  └─ Fills in Completion Report
│  └─ Exits
│
🐘 PA re-ingests
│  └─ Reads Completion Report
│  └─ Updates PROJECT.md ledger (WO4 ✓)
│  └─ Updates MEMORY.md (WO4/6 → WO5/6)
│  └─ Branches WO5 off WO4
│  └─ Repeats...
```

---

## License

See [LICENSE](LICENSE) for details.
