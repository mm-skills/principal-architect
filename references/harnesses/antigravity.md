---
name: harness-antigravity
description: Worker-spawning adapter for the Antigravity harness (harness-hosted runtime only — not for programmatic SDK agents). Selected when your sub-agent spawn tool is `invoke_subagent` (Google Antigravity).
---

# Harness Adapter — Antigravity

You are running under **Antigravity** (harness-hosted runtime — the CLI / chat interface, not a programmatic SDK agent). You confirmed this by self-inspection (Core Directive 6): your sub-agent spawn tool is `invoke_subagent`. This adapter governs how you spawn and right-size workers here.

> [!NOTE]
> This adapter applies to the **harness-hosted runtime** only. If you are building a programmatic agent via the Google Antigravity Python SDK, the `LocalAgentConfig` does expose a `model` field per agent — in that context, refer to the model-selection guidance in the SDK documentation instead.

## Spawn Mechanics

Spawn workers using the **`invoke_subagent`** tool with `TypeName: "self"` — this inherits the full harness configuration including tools, skills, and system prompt, making it the correct type for capable execution workers.

Key parameters to set on every Work Order spawn:

| Parameter | Value | Rationale |
|---|---|---|
| `TypeName` | `"self"` | Inherits full harness config; gives the worker all necessary tools. |
| `Role` | `"Execution Worker"` | Human-readable identity for the worker in the subagent list. |
| `Prompt` | bridge prompt from Step 4 | The full WORKORDER bridge prompt. |
| `Model` | See complexity mapping below | **Always set explicitly** — maps from the WORKORDER's `complexity` field. |
| `Workspace` | See decision table below | **Do not default to `"branch"` — read the decision table.** |

### `Workspace` Mode Decision

> [!CAUTION]
> **`"branch"` creates an isolated worktree at a path outside the user's permitted
> directory** (e.g. inside `~/.gemini/antigravity/brain/.../worktrees/`). The subagent
> will have **no permissions** for that path and will prompt the user for approval on
> every single file read or write. This defeats the purpose of autonomous execution.

| Scenario | Use | Rationale |
|---|---|---|
| **Single serialised worker** (most PRs — one WO at a time) | **`"inherit"`** | Worker operates in the same directory as the PA, which is already permitted. The branch is already checked out before spawning. No permission prompts. |
| **Truly parallel workers** on different branches simultaneously | **`"branch"`** | Gives each worker its own isolated worktree so they cannot clobber each other's staged changes. Only viable when the user has explicitly granted permissions to the worktree root path, or is prepared to approve access on first spawn. |

**The standard PA workflow (one WO at a time, branch pre-checked-out) always uses `"inherit"`.**
Use `"branch"` only when you have confirmed the user's permission grants cover the worktree path,
or when the user has explicitly requested parallel execution and accepted the permission prompts.

### Async / Auto-Wake Behaviour

`invoke_subagent` is **inherently asynchronous**. After spawning:
1. Inform the user the worker is running in the background.
2. **Drop control immediately** — do not poll or loop.
3. The system will **automatically wake you** when the worker sends a message via `send_message`. You will receive that message at the start of your next invocation.

Do not use `manage_subagents` to poll for completion. Wait to be woken.

## Artifact-Write Authorization (not needed here)

Unlike Claude Code, this runtime does **not** restrain workers from writing report-style `.md`
files. Workers author their own WORKORDER status updates, ADRs, graveyard notes, and
`@distillery` → `context_bridge.md` directly — this is the established pattern and the source of
the existing distillations in this workspace. No special authorization block in the spawn prompt
is required.

## Complexity → Model Mapping (The Efficiency Lever)

You, the Architect, run on **Pro** (or whatever model the user selected for the session) because you hold the macro context and make the judgment calls. Workers execute a strict, pre-specified checklist, so they should run on the **cheapest model that does the job without quality loss**. Model tier is *coupled to work-order specificity* — the tighter your `WORKORDER.md`, the cheaper the model that can execute it.

Map the WORKORDER's `complexity` field directly to the `Model` parameter on `invoke_subagent`:

| `complexity` | `Model` | Rationale |
|---|---|---|
| `mechanical` | `flash_lite` | Rote, deterministic work — fastest, cheapest tier. |
| `standard`   | `flash` | **Default worker tier.** Strong at code, fast, far cheaper than Pro. |
| `architect`  | `pro` | Reserve for WOs that genuinely need architect-grade reasoning. If you reach for this, first ask whether the ambiguity belongs back in *your* court before handoff. |

The `Model` you pass MUST match the `complexity` recorded in the WORKORDER frontmatter, so the choice stays auditable in the durable artifact.

### Advanced: Custom Agent Types via `define_subagent`

For tighter control on mechanical WOs, consider using `define_subagent` to create a stripped-down worker type with restricted tools (`enable_write_tools: true`, `enable_mcp_tools: false`, `enable_subagent_tools: false`) and a minimal system prompt. Then invoke it with `Model: "flash_lite"`. This reduces token overhead and prevents the worker from accessing tools it doesn't need. This is optional — using `TypeName: "self"` with the appropriate `Model` tier is the standard pattern.
