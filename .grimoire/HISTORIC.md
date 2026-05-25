# HISTORIC

1. **002-parallel-execute** [planned] — Introduces a dependency-graph execution model for `grimoire-execute`: `grimoire-plan` emits per-step YAML frontmatter (`depends-on`, `touches`), `grimoire-execute` runs topologically-sorted waves with one sub-agent per step in an isolated git worktree, cherry-picks back in step-number order, and tears down every worktree unconditionally. Pages without frontmatter keep the legacy strict-serial path (pure opt-in).
2. **001-grimoire-note-skill** [finished] — Adds the `grimoire-note` skill for incremental, surgical edits to `.grimoire/PROJECT.md`'s `Key Conventions` and `Notes` sections, and amends `§ Project context` to formalize the dual-writer contract with `grimoire-init`.
