---
name: grimoire-plan
description: Plan a page with Grimoire — writes step files under .grimoire/pages/ and registers the page in HISTORIC.md.
argument-hint: Page description
---

**[Required Reading]**
1. Read `${CLAUDE_PLUGIN_ROOT}/GRIMOIRE-CONVENTIONS.md`. Its rules (§ TDD, § Sub-agent spawning, § Commits, § .grimoire/ layout, § Project context, § Historic, § Pause-point pattern) are load-bearing for this skill. **Note:** this skill is the writer for `HISTORIC.md` (bootstrap + append + rotate); execute only updates status.
2. If `.grimoire/PROJECT.md` exists in the project root, read it for project context. If it does not exist, proceed without it and suggest the user run `grimoire-init` once this task is complete.
3. If `.grimoire/HISTORIC.md` exists, read it for recent-page context. If older context seems relevant, browse `.grimoire/bag/historic/` (newest suffix first). Missing → proceed without it; this skill will bootstrap it in Phase 4.

**[Objective]**
I need you to plan the implementation of the following page:

$ARGUMENTS

**[Phase 1: Analysis & Clarification]**
- Analyze the current codebase architecture.
- Research and identify the best practices for this specific implementation.
- Pause per `§ Pause-point pattern` (grimoire-plan: unclear requirements): if any requirement is unclear or a critical architectural decision is needed, ask me clarifying questions before proceeding.

**[Phase 2: Planning, Scope & Context Management]**
- CRITICAL: You are entirely responsible for evaluating the complexity and scope of the requested page.
- You must actively assess the potential token usage and context window size required for the execution phase. Your primary goal is to maintain maximum context lucidity and prevent degradation during execution.
- **Output shape is always a page folder.** Per `§ .grimoire/ layout`, the page is saved as `.grimoire/pages/NNN-[page-name]/` containing one or more sequential step files (`1-[step-name].md`, `2-[step-name].md`, …):
  - **Simple page:** a single step file `1-[step-name].md` containing the full step-by-step execution plan.
  - **Larger page:** multiple sequential step files, each a smaller, highly cohesive, sequential portion of the work. Split when token usage or context degradation becomes a risk during execution.
- You choose `NNN` (project-wide chronological, next available) and the step count based on projected context lucidity per step.

**[Phase 3: Write the Page]**
- Create `.grimoire/pages/NNN-[page-name]/` if it does not exist.
- Write each step file as `1-[step-name].md`, `2-[step-name].md`, … in numeric order.
- Commit the page files per `§ Commits` (one logical commit for the planning files; do not bundle the HISTORIC update — that goes in Phase 4).

**[Phase 4: HISTORIC.md Maintenance]**
Run this phase only after the page folder and step files are written. Follow `§ Historic` strictly.

- **If `.grimoire/HISTORIC.md` does not exist:** create it with the new page as entry `1.`:
  ```
  # HISTORIC

  1. **NNN-[page-name]** [planned] — <1–2 sentence description of what the page delivers>
  ```
- **If the entry for this page already exists** (rerun/replan): ensure its status is `[planned]` and refresh the description in place. Do not duplicate, do not move it.
- **If the entry does not exist and `HISTORIC.md` has fewer than 5 entries:** prepend the new entry as item `1.` with status `[planned]`, renumbering the previous ones.
- **If the entry does not exist and `HISTORIC.md` already has 5 entries:** rotate first per `§ Historic` (create `.grimoire/bag/historic/` if needed; move `HISTORIC.md` to `.grimoire/bag/historic/HISTORIC-N.md` where `N = max existing suffix + 1`, or `1` if empty), then write a fresh `HISTORIC.md` containing only the new entry as item `1.` with status `[planned]`.

**[Phase 5: Historic Commit]**
- Commit the HISTORIC change per `§ Commits` as a separate commit: `chore: register page <name> in historic`.
- If a rotation also happened, the same commit covers both the rotated `HISTORIC-N.md` file and the fresh `HISTORIC.md`.
