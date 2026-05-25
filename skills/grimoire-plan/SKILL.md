---
name: grimoire-plan
description: Plan a page with Grimoire — reads SPEC.md and writes sequential step files under the page folder.
argument-hint: Page number (e.g. 42)
---

**[Required Reading]**
1. Read `${CLAUDE_PLUGIN_ROOT}/GRIMOIRE-CONVENTIONS.md`. Its rules (§ TDD, § Sub-agent spawning, § Parallel execution, § Commits, § .grimoire/ layout, § Project context, § Historic, § Pause-point pattern) are load-bearing for this skill. **Note:** this skill only **updates the page's `HISTORIC.md` entry status** from `[spec]` to `[planned]` in place — it never bootstraps, appends, or rotates. `HISTORIC.md` ownership (bootstrap + append + rotate) belongs to `grimoire-spec`. This skill requires the page to already have a `SPEC.md` and a `[spec]` entry in `HISTORIC.md`; if either is missing, hard-stop per `§ Pause-point pattern` (grimoire-plan: precondition check).
2. If `.grimoire/PROJECT.md` exists in the project root, read it for project context. If it does not exist, proceed without it and suggest the user run `grimoire-init` once this task is complete.
3. If `.grimoire/HISTORIC.md` exists, read it for recent-page context. If older context seems relevant, browse `.grimoire/bag/historic/` (newest suffix first). **Missing → hard-stop**, because the page-status check below requires it.

**[Objective]**
Plan the implementation of the page identified by:

$ARGUMENTS

**[Phase 1: Page Resolution & Precondition Check (HARD STOP)]**
Per `§ Pause-point pattern` (grimoire-plan: precondition check). Run these checks **before** doing anything else; STOP on the first failure with the literal message shown.

1. **Parse the page number.** Accept `$ARGUMENTS` as a bare integer with or without zero-padding (`42` or `042`); normalize to `NNN` (3-digit zero-padded). Reject anything that is not parseable as an integer.
2. **Locate the page folder.** Glob `.grimoire/pages/NNN-*/`. Exactly one match expected.
   - No match → `❌ Page NNN não existe. Rode /grimoire-spec primeiro.` and STOP.
3. **Confirm `SPEC.md`.** Verify `.grimoire/pages/NNN-[page-name]/SPEC.md` exists and is non-empty.
   - Missing → `❌ Page NNN não tem SPEC.md. Rode /grimoire-spec primeiro.` and STOP.
4. **Check HISTORIC status.** Read `.grimoire/HISTORIC.md` and find the entry whose `<page-name>` matches the folder.
   - Entry missing (rotated or never registered) → `❌ Page NNN não está registrada no HISTORIC. Rode /grimoire-spec primeiro.` and STOP.
   - Status `[planned]` → `❌ Page NNN já foi planejada.` and STOP.
   - Status `[finished]` → `❌ Page NNN já foi finalizada.` and STOP.
   - Status `[spec]` → proceed.
5. **Load context.** Read `SPEC.md` in full; re-confirm `PROJECT.md` context is loaded.

Do **not** offer to run any other skill automatically, and do **not** silently fix the state. The user is in control.

**[Phase 2: Analysis & Clarification]**
- Analyze the current codebase architecture relative to what `SPEC.md` requires.
- Research and identify the best practices for this specific implementation.
- Pause per `§ Pause-point pattern` (grimoire-plan: unclear requirements): if any acceptance criterion is still ambiguous, or a critical architectural decision is needed that the SPEC did not lock in, ask clarifying questions before writing step files. Open questions noted in `SPEC.md` are the natural starting point.

**[Phase 3: Planning, Scope & Context Management]**
- CRITICAL: You are entirely responsible for evaluating the complexity and scope of the page.
- Actively assess potential token usage and context window size required for the execution phase. Your primary goal is to maintain maximum context lucidity and prevent degradation during execution.
- **Output shape is always a set of sequential step files in the existing page folder.** Per `§ .grimoire/ layout`:
  - **Simple page:** a single step file `1-[step-name].md` containing the full step-by-step execution plan.
  - **Larger page:** multiple sequential step files, each a smaller, highly cohesive, sequential portion of the work. Split when token usage or context degradation becomes a risk during execution.
- The page folder and its `NNN-[page-name]` identity already exist — do not rename, do not move, do not create a new page folder. Choose only the step count and the per-step names.
- **Inter-step dependencies (opt-in).** When the planned step breakdown contains steps that are **genuinely independent of each other** (different files, no logical ordering needed), the planner MAY emit YAML frontmatter on each step file declaring `depends-on: [<step-numbers>]` and `touches: [<paths>]` per `§ Parallel execution`. This unlocks parallel-wave execution in `grimoire-execute`. Do not restate the schema or wave rules — they live in `§ Parallel execution`.
- **When NOT to emit frontmatter.** If every step is genuinely sequential — each builds on the prior step's edits in a load-bearing way — the planner SHOULD emit no frontmatter. The page then runs via the legacy strict-serial path, which is the default. Emitting frontmatter where it isn't warranted just adds noise.
- **DAG construction and validation (before writing step files).** When frontmatter is going to be emitted, the planner MUST, before writing any step file:
  1. Build the DAG: nodes = step files, edges = `depends-on` references.
  2. **Detect cycles.** Any cycle is a hard error — the planner MUST NOT write step files with a cyclic DAG. If the natural step breakdown produces a cycle, revise the breakdown until the DAG is acyclic, or abandon frontmatter and fall back to legacy serial.
  3. **Check for dangling references.** Every step number referenced in any `depends-on` must correspond to an actual planned step. Dangling references are a hard error.
  4. **Compute waves** (Kahn-style topological levels per `§ Parallel execution`): wave 0 = all roots; wave K = all steps whose `depends-on` lies entirely in waves `< K`.
  5. **Advisory `touches`-overlap check.** When two sibling steps in the same wave declare overlapping `touches` paths, surface the overlap as a warning (not a hard error) — the user decides at the pause-point whether to keep them parallel or force a serial edge.
- Before proceeding to Phase 4, perform the final clarity check per `§ Pause-point pattern` (grimoire-plan: final clarity check): self-review the planned step breakdown and, if any architectural choice, step boundary, or tradeoff is still based on a silent assumption, PAUSE and ask the user via `AskUserQuestion` (or equivalent available tool). Only continue to Phase 4 once the plan is fully unambiguous.
- **DAG presentation at the final clarity check.** When `depends-on` frontmatter will be emitted on one or more step files, the final clarity check MUST also present the full DAG to the user before any step file is written:
  - Each step number, its planned name, and its `depends-on` set.
  - The computed waves (e.g., `Wave 0: {1}`, `Wave 1: {2, 3}`, `Wave 2: {4, 5}`).
  - Any advisory `touches`-overlap warnings.

  The user MUST be able to veto the parallelization (force serial / change deps / change step boundaries) before any step file is written. If the user vetoes, fall back to legacy serial (no frontmatter) or re-plan per the user's correction. If no frontmatter will be emitted, the existing final-clarity-check semantics apply unchanged.

**[Phase 4: Write Step Files]**
- Inside the existing `.grimoire/pages/NNN-[page-name]/` folder, write each step file as `1-[step-name].md`, `2-[step-name].md`, … in numeric order. Do not touch `SPEC.md`.
- When the DAG approved at the final clarity check requires frontmatter, prepend each step file with a YAML frontmatter block declaring `depends-on` and (recommended) `touches`, per the schema in `§ Parallel execution`. Steps that are DAG roots use `depends-on: []` (or omit it — both are valid). The markdown body of the step file remains the canonical instructions for the sub-agent.

  Illustrative example (not normative — schema lives in `§ Parallel execution`):

  ```
  ---
  depends-on: [1]
  touches:
    - skills/grimoire-execute/SKILL.md
  ---
  # Step 3 — Update grimoire-execute …
  ```
- Commit the step files per `§ Commits` (one logical commit per step file, or one commit for all step files if they are tightly coupled; do not bundle the HISTORIC update — that goes in Phase 6).

**[Phase 5: HISTORIC Status Update]**
Per `§ Historic`: status update only. **Never bootstrap, append, or rotate.**

- Find the entry in `.grimoire/HISTORIC.md` whose `<page-name>` matches the page just planned.
- Update its status from `[spec]` to `[planned]` **in place**. Do not change its position, do not append a new entry, do not rotate, do not touch any other entry.

**[Phase 6: Historic Commit]**
- Commit the HISTORIC change per `§ Commits` as a separate commit: `chore: mark page <name> planned`.
- End the run by telling the user the next step: `/grimoire-execute NNN`.
