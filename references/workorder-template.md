---
name: workorder-template
description: Canonical WORKORDER.md template for worker handoff. Read this when constructing a new WORKORDER at Step 3.
---

# WORKORDER Template

Every WORKORDER.md must follow this structure. Copy the sections below into the new file, filling in the bracketed placeholders.

## YAML Frontmatter (required)

```yaml
---
name: [project]-wo[Y]
description: Handoff instructions for Work Order [Y]
complexity: standard   # mechanical | standard | architect
---
```

The `complexity` field is mandatory and describes *the work*, not any specific model. It is harness-neutral; your loaded adapter (Core Directive 6) decides what — if anything — it maps to:
- **`mechanical`** — rote, near-zero judgment (mass rename, find/replace, doc/index regeneration, scaffolding boilerplate, run-verify-and-report).
- **`standard`** — the default: a clear boundary box, a known set of files, a concrete checklist.
- **`architect`** — genuinely requires high-end reasoning (cross-cutting refactor, design-sensitive code, irreducibly ambiguous spec). Selecting this is also a smell that the WO may still contain a decision *you* should resolve before handoff.

## Required Sections

### Role Definition
> You are an expert senior software engineer and execution-focused worker agent. You are NOT the Principal Architect. Do not make high-level architectural decisions, do not branch, and do not plan. Your task is strictly limited to executing the checklist below.

### Objective
1-2 sentence core intent of this Work Order.

### Strict Boundary Box
Explicit rules on what the agent should NOT touch. List files, directories, and concerns that are out of scope.

### Tooling Constraints
Before drafting, self-inspect your environment's available MCP servers. If relevant build, test, or lint tools exist, explicitly name them here and instruct the worker to use those specific MCP tools rather than raw CLI commands.

### Line-Item Instructions
Checkboxes detailing the exact files to modify:
- [ ] `path/to/file.ts` — description of change
- [ ] `path/to/other.ts` — description of change

### Verification Requirement
The worker must verify it compiles/runs according to the Tooling Constraints. **CRITICAL:** Before committing, the worker MUST ask the user if they are happy with the changes.

### Issue Surfacing
If you discover a defect, upstream bug, or concern *outside your boundary box*, surface it in your Completion Report below — do not file issues directly. The Principal Architect will triage and file them under the correct author identity.

---

## Completion Report (written by the worker at task completion)

Append this section to the WORKORDER.md upon completing the checklist.

```markdown
---
## Completion Report
<!-- Written by the worker at task completion -->

### Status
complete | partial | blocked

### Changes Made
- `path/to/file` — what changed and why

### Decisions Made
<!-- Only non-obvious decisions. Omit this section entirely for mechanical WOs. -->
- Decision description and rationale

### Dead Ends & Warnings
<!-- Things that didn't work. Critical for preventing future agents from repeating mistakes. -->
- What was attempted, why it failed, and what to avoid

### Out-of-Scope Findings
<!-- Defects, bugs, or concerns outside the boundary box. The PA will triage these. -->
- Finding description and suggested action

### Blockers
<!-- Empty if status is 'complete'. If partial/blocked, explain what prevents completion. -->
(none)
```
