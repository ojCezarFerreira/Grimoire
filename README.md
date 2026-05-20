# Sage

A Claude Code plugin bundling three skills for a disciplined **Plan → Execute** pipeline with a **Quick** fast-path.

- `/sage-plan <feature>` — analyze the codebase, ask clarifying questions, then write a plan file into `.planning/`. Splits into sub-plans automatically when the scope is large enough to threaten context lucidity.
- `/sage-execute <path>` — execute a plan (single file or sub-plan directory) via sub-agents, one per sub-plan in strict numeric order. On success, moves the plan(s) into `.planning/finished/`.
- `/sage-quick <fix>` — fast-path for small fixes. Has a scope gatekeeper (stops and redirects you to `/sage-plan` if the task is too big) and requires explicit authorization of the inline plan before any code is written.

All three skills share a single source of truth for the workflow rules — strict TDD, atomic Conventional Commits, sub-agent orchestration, and the `.planning/` directory layout — in [SAGE-CONVENTIONS.md](SAGE-CONVENTIONS.md).

## Install

Add this repo as a marketplace, then install the plugin:

```
/plugin marketplace add <github-owner>/sage
/plugin install sage
```

Replace `<github-owner>/sage` with the repo path (for example `cezarleferr/sage`). Other install options:

```
# Install to project scope (shared via .claude/settings.json)
claude plugin install sage --scope project

# Local trial without a marketplace
claude --plugin-dir /path/to/this/repo
```

After install, the three skills are available as `/sage-plan`, `/sage-execute`, `/sage-quick` in any session.

## Workflow

1. `/sage-plan "add user-auth endpoint"` → produces `.planning/001-add-user-auth-endpoint.md` (or a `.planning/001-add-user-auth-endpoint/` folder of sub-plans).
2. `/sage-execute .planning/001-add-user-auth-endpoint.md` → runs the plan via a sub-agent under strict TDD, commits atomically, then moves the plan to `.planning/finished/`.
3. Use `/sage-quick "fix typo in login error message"` for trivial fixes — Sage will stop you and route you back to `/sage-plan` if the request is too large.

## Editing the plugin

See [CLAUDE.md](CLAUDE.md) for maintainer notes.
