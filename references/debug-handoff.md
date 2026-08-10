---
name: debug-handoff
description: Interactive testing and debug handoff protocol. Read this when a Work Order requires human-in-the-loop testing, or when the PA detects it is being pulled into tactical debugging mid-session.
---

# Debug / Interactive Testing Handoff Protocol

This protocol preserves the Principal Architect's strategic context by delegating interactive testing and debugging to a dedicated worker session. The PA packages all context into a bridge prompt rather than getting pulled into tactical execution.

## When to Trigger

### Proactive (during WO planning)

Trigger the interactive handoff when decomposing a WO that inherently requires human-in-the-loop iteration:

- **UI/UX verification** — the user needs to visually inspect rendering, layouts, animations
- **Integration testing** — requires running local services, database seeding, manual endpoint testing
- **Device/browser testing** — changes need verification across environments the PA cannot access
- **Exploratory QA** — the scope of what to test is not fully specified up front
- **User acceptance** — the user wants to interact with a feature before signing off

In these cases, set `complexity: interactive` in the WORKORDER frontmatter from the start.

### Reactive (mid-session detection)

Trigger the interactive handoff when you recognise the PA is being pulled into debugging:

**Signals to watch for:**
- The user describes symptoms, errors, or unexpected behavior and asks the PA to investigate
- The conversation shifts from architecture/planning to adding `console.log`, inspecting stack traces, or tweaking CSS
- The PA finds itself writing debugging or diagnostic code rather than planning
- The user asks "can you try..." or "what if we change..." in a tight iteration loop
- Multiple back-and-forth exchanges on the same tactical issue without architectural decisions being made

**When you detect these signals:**
1. **Stop immediately.** Do not continue debugging.
2. Inform the user: *"This is moving into interactive debugging territory. Let me package what we know into a debug work order so you can work through this with a fresh worker session — that way we preserve our architectural context here."*
3. Proceed with the handoff protocol below.

## The Handoff Protocol

### Step 1: Create the Debug WO Folder

Create the folder within the existing project boundary:
`.agents/skills/principal-architect-workspace/pr-[project-name]/wo[Y]-debug-[topic]/`

Use the next available WO number in the ledger. The `debug-` prefix in the topic makes the purpose immediately clear.

### Step 2: Write the Debug WORKORDER

The debug WORKORDER uses the same canonical template (`references/workorder-template.md`) with these adaptations:

**YAML Frontmatter:**
```yaml
---
name: [project]-wo[Y]
description: Interactive debug/testing session for [topic]
complexity: interactive
---
```

**Role Definition (adapted for interactive work):**
> You are an expert senior software engineer and debugging-focused worker agent. You are NOT the Principal Architect. Do not make high-level architectural decisions, do not branch, and do not plan beyond the immediate debugging scope. Your task is to work interactively with the user to resolve the issue described below.

**Required additional sections (beyond the standard template):**

#### Reproduction Context
What the user was doing, what they expected, and what actually happened. Include:
- Steps to reproduce
- Expected vs. actual behavior
- Environment details (browser, OS, Node version, etc.) if relevant

#### Known Symptoms
Concrete error messages, stack traces, screenshots, or behavioral descriptions.

#### Relevant File Paths
Files the PA has identified as likely involved. Be specific — the debug worker should not need to search the whole codebase.

#### What Has Already Been Tried
Critical for preventing the debug worker from repeating the PA's (or user's) prior failed approaches.

#### Verification Requirement (adapted)
The standard verification section should be replaced with:
> Work interactively with the user to diagnose and resolve the issue. The session is complete when the user confirms the issue is resolved, or when you have exhausted your investigation and documented your findings. Before wrapping up, ask the user to confirm they are satisfied.

### Step 3: Provide the Bridge Prompt to the User

**Do NOT spawn an autonomous subagent.** Interactive WOs are user-driven. Instead, present the user with a formatted bridge prompt they can paste into a new session:

```markdown
## 🔧 Debug Session — [topic]

I've prepared a debug work order with full context at:
`[relative path to WORKORDER.md]`

**Start a new session and paste this prompt:**

---

# [project-name] / WO[Y] — Debug: [topic]

You are an expert senior software engineer and debugging-focused worker agent.
**DO NOT trigger or act as the principal-architect.**

Read your instructions and full context here via view_file:
`.agents/skills/principal-architect-workspace/pr-[project-name]/wo[Y]-debug-[topic]/WORKORDER.md`

Work interactively with the user to resolve the issue. When the issue is resolved
(or investigation is exhausted), do the following:

1. Append your Completion Report to the WORKORDER.md (template is at the bottom)
2. Present the user with this return prompt for the Principal Architect:

> **Return to PA session:** "The debug worker has completed WO[Y] (debug-[topic]).
> Read the Completion Report at `.agents/skills/principal-architect-workspace/pr-[project-name]/wo[Y]-debug-[topic]/WORKORDER.md`
> and update the project status."

---

I'll resume when you return with the worker's findings.
```

### Step 4: Update PROJECT.md and Drop Control

1. Add the debug WO to the PROJECT.md ledger (marked as `[ ]` — in progress with user)
2. Inform the user you are dropping control and will resume when they return
3. **Stop.** Do not continue planning or executing other WOs — the debug WO may affect subsequent work.

## Return Protocol

When the user returns from the debug session:

1. **Read the debug WORKORDER's Completion Report** — assess whether the issue was resolved, what was learned, and any out-of-scope findings
2. Re-ingest `PROJECT.md` and `memory/MEMORY.md` to refresh state
3. Update the WO ledger: mark the debug WO as complete
4. Enrich the Session Log with any project-wide findings from the debug session (per `references/distillation-guide.md`)
5. Resume the normal workflow — loop back to the next planned WO

## Harness-Specific Notes

The interactive handoff is **harness-agnostic by design** — the user always drives the debug session in a new conversation. However:

- **Antigravity**: The user starts a new Antigravity session. No `invoke_subagent` — the session is standalone.
- **Claude Code**: The `spawn_task` pattern is a natural fit — it surfaces a chip for the user to open. Alternatively, the PA can provide a bridge prompt for a fully manual new session.
- **Generic**: Bridge prompt is the only pattern. Present it to the user and wait for their return.
