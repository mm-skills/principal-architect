---
name: distillation-guide
description: Tiered inline distillation formats. Read this when writing or reviewing completion epilogues and session logs.
---

# Distillation Guide — Tiered Inline Enrichment

This guide defines how knowledge is captured at each tier of the architecture. The goal is to prevent cold-start amnesia: a future agent resuming work must find decisions, dead-ends, and outcomes in the documents it already reads — not in separate files it might miss.

## Theory of Mind

If you do not explicitly document dead-ends and failed attempts, the next agent will confidently waste tokens repeating the exact same errors. The most valuable knowledge is often *what didn't work and why* — not just what did.

## Tier 3: Worker → WORKORDER.md Completion Report

Workers append a structured Completion Report directly to their WORKORDER.md upon task completion. See `references/workorder-template.md` for the exact format.

The Completion Report is the worker's primary output beyond the code changes themselves. It captures:
- **Status**: Did the work complete, partially complete, or block?
- **Changes Made**: What files were modified and why (the commit message is not enough — context matters)
- **Decisions Made**: Non-obvious choices the worker made within their boundary box
- **Dead Ends**: Failed approaches, with enough detail to prevent repetition
- **Out-of-Scope Findings**: Issues surfaced for the PA to triage

## Tier 2: PA → PROJECT.md Session Log

The PA progressively enriches PROJECT.md with a session log section. This is written during Step 5 (re-ingestion) when the PA reads the worker's Completion Report, and during Step 6 (wrap-up).

```markdown
## Session Log
<!-- Progressively enriched by the PA as WOs complete -->

### [ISO timestamp] — WO[Y] Complete: [topic]
**Summary**: [1-2 sentence outcome]
**Cross-cutting decisions**: [decisions affecting subsequent WOs or project architecture]
**Surfaced findings**: [out-of-scope items flagged by the worker — per Directive 8]
**Dead ends** (project-wide): [only dead ends with project-wide implications — don't duplicate mechanical ones from the WORKORDER]

### [ISO timestamp] — Project Wrap-Up
**Architectural outcome**: [what changed at the system level, not file level]
**Lessons for future projects**: [cross-project learnings]
**Unresolved items**: [anything deferred, flagged, or left incomplete]
```

### Depth Calibration

Not every worker dead-end belongs in the session log. Use your judgment:
- **Escalate to session log**: Dead ends involving dependencies, API limitations, architectural constraints, or anything that affects other WOs or future projects.
- **Leave in WORKORDER**: Dead ends involving local implementation choices (e.g., "tried approach X for this function, it was slower").

The progressive-reveal principle applies: the session log is a summary. A future PA reads it first, then drills into specific WORKORDER Completion Reports only if deeper detail is needed.

## Tier 1: Chief Architect → PROGRAM.md Session Log

The Chief Architect enriches PROGRAM.md with a programme-level session log when a PA completes a project. The Chief reads the completed PA's PROJECT.md session log (not a separate context_bridge.md) and consolidates:

```markdown
## Programme Session Log

### [ISO timestamp] — PR-[N] Complete: [project-name]
**Summary**: [1-2 sentence outcome]
**Cross-PR decisions**: [decisions affecting the dependency graph or subsequent PRs]
**Architecture evolution**: [how the system-level architecture changed]
**Unresolved items escalated from PA**: [items the PA flagged for programme-level attention]
```

## Legacy Compatibility

If a `context_bridge.md` exists in a project directory from a previous distillery invocation, read it as legacy context. Future sessions should use the inline session log pattern instead.
