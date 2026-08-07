---
name: harness-claude-code
description: Worker-spawning adapter for the Claude Code harness (Anthropic CLI / Claude Agent SDK). Selected when your spawn tool can override the worker model per spawn.
---

# Harness Adapter — Claude Code

You are running under **Claude Code** (Anthropic's CLI / Claude Agent SDK). You confirmed this by self-inspection (Core Directive 6): your sub-agent spawn tool exposes a per-spawn **`model`** parameter. This adapter governs how you spawn and right-size workers.

## Spawn Mechanics

Spawn workers with the **`Agent`** tool:
- `subagent_type: general-purpose` for execution workers.
- `model:` — **always set this explicitly.** Never let a worker silently inherit the Architect's Opus context; that is the single most expensive mistake in this workflow.
- `run_in_background: true` for autonomous execution — the worker runs unattended and auto-wakes you on completion (the default for well-specified Work Orders).
- `prompt:` — the bridge prompt from Step 4 of the skill.

### Autonomous vs. interactive (choose per Work Order)
There are two spawn patterns; pick per WO rather than committing to one default:
- **Autonomous `Agent` worker** — `run_in_background: true`, reports back only to you (the Architect), auto-wakes you. Best for mechanical, well-specified orders.
- **Interactive `spawn_task` worker** — surfaces a chip that opens a separate session + its own worktree for the user to drive directly through a few iterations. Best when hands-on iteration is valuable. Does **not** auto-report back to you.

If the WO benefits from human-in-the-loop iteration, briefly surface this choice to the user before spawning.

## Artifact-Write Authorization (REQUIRED for worker write-backs)

On Claude Code, spawned subagents carry a built-in guideline against creating
report/summary/findings/analysis `.md` files. It is a **soft** guideline — *verified*: there is no
hook or permission blocking such writes, and a worker given an explicit override writes them
cleanly. But a worker handed a WORKORDER that merely *names* a deliverable file (e.g. `FINDINGS.md`)
will typically **over-refuse** and return its findings as text instead, because its system-prompt
guideline outweighs an instruction it only read from a file.

This matters because the PA workflow relies on workers authoring durable memory artifacts —
WORKORDER status updates, ADRs, graveyard notes, and `@distillery` → `context_bridge.md`. To make
worker write-backs reliable, you **MUST** include this authorization block **verbatim** in every
worker spawn prompt whose WORKORDER names a file deliverable:

> **Artifact-write authorization:** Writing the artifact(s) named in your WORKORDER —
> e.g. `FINDINGS.md`, ADRs, graveyard notes, WORKORDER status updates, and any `@distillery`
> output (`context_bridge.md`) — is an explicit, user-approved deliverable. This OVERRIDES the
> general guideline about not creating report/summary/markdown files: that guideline targets
> *unsolicited* files, and yours are explicitly requested and are the point of the task. Do not
> substitute returning text for writing a file the WORKORDER names as the deliverable.

**Graceful degradation:** if you omit the block (or for a pure investigation WO), the worker safely
falls back to returning findings as text, and **you (the PA) capture them into the artifact
yourself** — a fully valid pattern.

**`@distillery` caveat (one hop removed):** the override is proven for a *direct* `Write`. It has
**not** been separately confirmed that `@distillery` invoked *inside* a subagent writes
`context_bridge.md` without balking. On first use, verify the worker's distillation actually lands
the file; if it doesn't, fall back to **running `@distillery` yourself** (PA-side) after the worker
returns.

## Complexity → Model Mapping (The Efficiency Lever)

You, the Architect, run on **Opus** because you hold the macro context and make the judgment calls. Workers execute a strict, pre-specified checklist, so they should run on the **cheapest model that does the job without quality loss**. Model tier is *coupled to work-order specificity* — the tighter your `WORKORDER.md`, the cheaper the model that can execute it. This is the primary cost/latency lever in the whole workflow.

Map the WORKORDER's `complexity` field directly to the `model` parameter:

| `complexity` | `model` | Rationale |
|---|---|---|
| `mechanical` | `haiku` | Rote, deterministic work — fastest, cheapest tier. |
| `standard`   | `sonnet` | **Default worker tier.** Strong at code, fast, far cheaper than Opus. |
| `architect`  | `opus` | Reserve for WOs that genuinely need architect-grade reasoning. If you reach for this, first ask whether the ambiguity belongs back in *your* court before handoff. |

The `model` you pass MUST match the `complexity` recorded in the WORKORDER frontmatter, so the choice stays auditable in the durable artifact.
