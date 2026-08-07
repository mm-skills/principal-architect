---
name: harness-generic
description: Fallback worker-spawning adapter for any harness not matched by a specific adapter. Selected when neither the claude-code nor antigravity adapters cleanly apply.
---

# Harness Adapter — Generic Fallback

You reached this adapter because self-inspection (Core Directive 6) did not cleanly match a known harness. Spawn workers using whatever sub-agent or task primitive your host provides, and proceed conservatively.

## Spawn Mechanics

Use your host's native mechanism to spawn a sub-agent with the role `Execution Worker`, passing the bridge prompt from Step 4 of the skill. If you have no spawn primitive at all, do not fabricate one — instead present the bridge prompt to the user and ask them to run the Work Order, then resume on their return.

## Complexity → Capability

Treat the WORKORDER's `complexity` field as **advisory only**: record it for portability and let it inform how tightly you scope the order, but do not assume any model/capability lever exists. Lean on work-order precision and, where useful, batching or serialization to control cost.

If you later determine your harness *does* expose a per-spawn model override, switch to `claude-code.md`; if it spawns but cannot override the model, switch to `antigravity.md`.
