---
name: grimoire-note
description: Surgically refine .grimoire/PROJECT.md's Key Conventions and Notes from a free-text input — semantic dedup, retroactive consolidation, IDE-aware diff review.
argument-hint: Free-text note(s) to fold into PROJECT.md's Key Conventions or Notes
---

**[Required Reading]**
1. Read `${CLAUDE_PLUGIN_ROOT}/GRIMOIRE-CONVENTIONS.md`. Only `§ Project context`, `§ IDE-aware review`, and `§ Commits` apply to this skill. `§ TDD`, `§ Sub-agent spawning`, `§ Historic`, `§ Pause-point pattern` (for pipeline pages), and `§ .grimoire/ layout` (for pages) do **not** apply — this skill is orthogonal to the Spec → Plan → Execute pipeline, never creates a page folder, never touches `HISTORIC.md`, and never spawns execute sub-agents.
2. If `.grimoire/PROJECT.md` exists in the project root, read it for project context — defer the hard-stop check to Phase 1 below so the user-facing message is presented cleanly.

**[Objective]**
Fold the user's free-text note(s) into `.grimoire/PROJECT.md` with surgical, incremental edits restricted to `## Key Conventions / Constraints` and `## Notes`. Never rewrite the whole file, never add a new section, never regenerate a skeleton — only sharpen what is already there or append a single well-phrased entry.

$ARGUMENTS

**[Hard constraints]**
- Only `.grimoire/PROJECT.md` may be edited. No other file in the repo is touched.
- Only `## Key Conventions / Constraints` and `## Notes` may be modified. Never add, rename, or remove any section header. Never edit `## Purpose`, `## Audience`, `## Tech Stack`, `## Repository Layout`, or `## Current Status`.
- Never create `.grimoire/PROJECT.md` if it is missing — hard-stop instead.
- Never create a page folder, never write to `.grimoire/HISTORIC.md`, never spawn execute sub-agents.

**[Phase 1: Precondition Check (HARD STOP)]**
Verify `.grimoire/PROJECT.md` exists at the project root. If it is missing, print the canonical English message verbatim:

> ❌ .grimoire/PROJECT.md does not exist.
> `grimoire-note` edits the `Key Conventions` and `Notes` sections of `PROJECT.md` — without the file there is nowhere to write.
> Run `/grimoire-init` first to create the base context, then re-invoke `/grimoire-note`.

The message may also be rendered in the user's interaction language when detected (e.g., pt-BR); English is the default and the canonical reference. STOP. Do **not** offer to run `grimoire-init` automatically. Do **not** create a skeleton.

**[Phase 2: Input Parsing & Semantic Split]**
Parse the user's free-text input. Semantically split it into N distinct rules when the input carries several (e.g., two sentences encoding two unrelated conventions). Single-rule inputs stay as one entry. Produce a structured list internally — the user-facing surface of the split happens at Phase 6 ("Detected N distinct rules: …").

**[Phase 3: Read & Synthesize Against Existing Sections]**
Read `## Key Conventions / Constraints` and `## Notes` in full from `.grimoire/PROJECT.md`. For each rule from Phase 2, run a semantic pass against every existing entry in both sections to detect:

- **Duplication** — the new rule already exists verbatim or in spirit.
- **Refinement** — the new rule sharpens or constrains an existing one.
- **Generalization** — the new rule subsumes (or is subsumed by) an existing one.
- **Contradiction** — the new rule conflicts with an existing one.

For each detected relationship, propose the best synthesis: merge, rewrite, generalize, or accept as new. **Minimum-word phrasing is an explicit objective** — favor terse, declarative entries over verbose ones, and prefer rewriting two near-duplicate lines into one over appending a third.

**[Phase 4: Retroactive Consolidation Pass]**
Independently of the new input, scan all existing rules in both sections and propose rewrites where synthesis can be improved: consolidate near-duplicates already on the page, tighten verbose phrasings, merge adjacent rules that say the same thing. This pass runs **every invocation**, not just when the new input triggers it — the cost is acceptable because the synthesis discipline keeps the file naturally small.

**[Phase 5: Section Assignment]**
Assign each accepted rule to its section using this heuristic:

- **`## Key Conventions / Constraints`** — normative / hard / "must" rules. Signals: `always`, `never`, `must`, `enforce`, "single source of truth", commit-message patterns, language rules.
- **`## Notes`** — observational / contextual / state. Signals: "the runtime currently announces X", "this repo dogfoods itself", "some operator messages are pt-BR by design".

When a rule is genuinely ambiguous — hedges like "tends to", "usually", "often" — ask the user via the available question tooling (e.g., `AskUserQuestion`) which section it belongs to before proposing the diff.

**Misfit handling.** When the rule fits neither section's intent (e.g., it would belong to `## Tech Stack` or `## Repository Layout` in spirit), force-fit into the closest section (default: `## Notes`) and flag the misfit explicitly in the Phase 6 diff narration so the user can override. **Never create a new section** — that power stays with `grimoire-init`.

**[Phase 6: IDE-Aware Review]**
Per `§ IDE-aware review`:

- **IDE detected:** write the proposed `PROJECT.md` content directly to `.grimoire/PROJECT.md` (its final destination — same pattern as `grimoire-init`). Tell the user in one short line that the draft is open and what answer you're waiting for, e.g. `📄 PROJECT.md atualizado em .grimoire/PROJECT.md — revise no editor e diga "ok" para seguir, ou liste mudanças.` (Operator-facing message may be pt-BR per project precedent.)
- **Terminal-only fallback:** present the diff inline in chat.

When applicable, surface the Phase 2 split as a short prose narration at this checkpoint: `Detected N distinct rules: …`. Also surface any misfit force-fits from Phase 5 so the user can override the section assignment.

PAUSE per `§ Pause-point pattern`. Wait for explicit user approval, requested edits, or abandonment.

- **On approval:** proceed to Phase 7.
- **On requested edits:** re-apply Phases 3–5 with the user's corrections and re-pause.
- **On abandonment:** restore `.grimoire/PROJECT.md` to its pre-edit state by reading `git show HEAD:.grimoire/PROJECT.md` and writing it back — never leave an unapproved file on disk.

**[Phase 7: Commit]**
Per `§ Commits`:

- One atomic commit per invocation, message exactly: `docs(grimoire): refine project context`.
- The commit must touch **only** `.grimoire/PROJECT.md`. If any other file is dirty in the working tree, do not include it in this commit — warn the user instead and let them stage/commit those changes separately.

**[Style / discipline guardrails]**
- Operator-facing messages default to English; pt-BR is permitted per project precedent (existing skills' hard-stop messages). Commit messages and SPEC-style content stay English.
- **Single source of truth.** The dual-writer contract between `grimoire-init` and `grimoire-note` lives in `GRIMOIRE-CONVENTIONS.md § Project context` — do not duplicate it here.
- **No `[Required Reading]` change** in any other skill is required: `.grimoire/PROJECT.md` is already auto-loaded via `§ Project context`.
