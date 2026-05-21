# Grimoire

A Claude Code plugin bundling six skills for a disciplined **Spec → Plan → Execute** pipeline with a **Quick** fast-path, an **Init** step that gives every session shared project context, and an **Update** step that keeps the plugin itself current.

Every change tracked by Grimoire is called a **page**: a folder under `.grimoire/pages/` containing a `SPEC.md` and one or more sequential step files, plus a single entry in `.grimoire/HISTORIC.md` that records its status (`[spec]` → `[planned]` → `[finished]`).

- `/grimoire-init` — analyze the project and interview you to produce `.grimoire/PROJECT.md` (purpose, audience, tech stack, conventions). Every other Grimoire skill loads that file on startup so the orchestrator and its sub-agents share baseline context. Re-running enters update mode and asks only about deltas.
- `/grimoire-spec <request>` — analyze the codebase, ask clarifying questions until consensus, then write the page as `.grimoire/pages/NNN-[page-name]/SPEC.md`. Bootstraps/appends/rotates `.grimoire/HISTORIC.md`, registering the page with status `[spec]`. This is the only skill that creates a new page.
- `/grimoire-plan <NNN>` — read the page's `SPEC.md` and write sequential step files (`1-[step].md`, `2-[step].md`, …) into the existing page folder. Updates the page's `HISTORIC.md` entry from `[spec]` to `[planned]`. Hard-stops if the page does not exist, has no `SPEC.md`, or is not in `[spec]` status.
- `/grimoire-execute <NNN>` — execute the page's step files via sub-agents, one per step file in strict numeric order. On success, updates the page's `HISTORIC.md` entry to `[finished]` in place — no files are moved. Hard-stops if the page is not in `[planned]` status or has no step files.
- `/grimoire-quick <fix>` — fast-path for small fixes. Has a scope gatekeeper (stops and redirects you to `/grimoire-spec` if the task is too big) and requires explicit authorization of the inline plan before any code is written. Stays ephemeral: no page folder, no `HISTORIC.md` entry.
- `/grimoire-update` — keep the plugin itself up to date. Reads the installed version, fetches the latest from GitHub, shows you exactly what changed in the changelog between the two, and only then guides you through the two official Claude Code commands (`/plugin marketplace update grimoire` + `/reload-plugins`). Exits silently if you're already on the latest version. Writes nothing to your project.

The pipeline skills (`init`, `spec`, `plan`, `execute`, `quick`) share a single source of truth for the workflow rules — strict TDD, atomic Conventional Commits, sub-agent orchestration, the `.grimoire/pages/` layout, project context loading, and the `HISTORIC.md` recency log and status-of-record — in [GRIMOIRE-CONVENTIONS.md](GRIMOIRE-CONVENTIONS.md). `grimoire-update` is plugin self-maintenance and does not participate in those rules.

## Install

Add this repo as a marketplace, then install the plugin:

```
/plugin marketplace add ojCezarFerreira/grimoire
/plugin install grimoire@grimoire
```

The `grimoire@grimoire` form is `plugin-name@marketplace-name` (both happen to be `grimoire` here). Other install options:

```
# Install to project scope (shared via .claude/settings.json)
claude plugin install grimoire --scope project

# Local trial without a marketplace
claude --plugin-dir /path/to/this/repo
```

After install, the six skills are available as `/grimoire-init`, `/grimoire-spec`, `/grimoire-plan`, `/grimoire-execute`, `/grimoire-quick`, `/grimoire-update` in any session.

To pull a newer version later, just run `/grimoire-update` — it handles the version check, changelog diff, and the two official commands for you.

## Workflow

1. `/grimoire-init` (once per project) → produces `.grimoire/PROJECT.md` from a short interview; every subsequent skill loads it automatically. Re-run it later in update mode whenever the project's purpose, stack, or constraints have shifted.
2. `/grimoire-spec "add user-auth endpoint"` → after a clarifying interview, produces `.grimoire/pages/001-add-user-auth-endpoint/SPEC.md` and registers the page in `.grimoire/HISTORIC.md` as `1. **001-add-user-auth-endpoint** [spec] — …` (rotating to `.grimoire/bag/historic/` if the log was already full).
3. `/grimoire-plan 1` → reads `SPEC.md`, plans the implementation, and writes sequential step files (e.g., `1-schema.md`, `2-endpoints.md`, `3-tests.md`) into the same page folder. Updates the entry to `[planned]` in place.
4. `/grimoire-execute 1` → runs each step file via a sub-agent under strict TDD, commits atomically, then updates the page's `HISTORIC.md` entry in place to `[finished]`. The page folder stays in `.grimoire/pages/`.
5. Use `/grimoire-quick "fix typo in login error message"` for trivial fixes — Grimoire will stop you and route you back to `/grimoire-spec` if the request is too large.

The page folder ends up looking like this:

```
.grimoire/pages/001-add-user-auth-endpoint/
├── SPEC.md          ← grimoire-spec
├── 1-schema.md      ← grimoire-plan
├── 2-endpoints.md   ← grimoire-plan
└── 3-tests.md       ← grimoire-plan
```

Page numbers passed to `/grimoire-plan` and `/grimoire-execute` accept either bare integers (`1`, `42`) or zero-padded form (`001`, `042`). If a skill is invoked out of order (plan before spec, execute before plan, or re-running a finished page), it hard-stops with a clear message naming the current state — it never silently fixes the state or runs another skill for you.

## Editing the plugin

See [CLAUDE.md](CLAUDE.md) for maintainer notes.
