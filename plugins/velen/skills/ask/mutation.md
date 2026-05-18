---
name: velen-cli-mutation
description: Use when you need to reason about future mutating Velen CLI flows or local state changes such as auth/org selection without assuming data-plane write access.
---

# Velen CLI Mutation

This leaf skill extends the main `SKILL.md` in this directory.

## Guardrails

- Follow the main `SKILL.md` before running local state mutations.
- Do not assume write access to remote data-plane resources through the public CLI surface except for explicit user-requested Knowledge Graph memory commands.
- Prefer explicit confirmation before changing persistent local CLI state.
- Treat `velen memory dataset delete` as destructive: use it only when the user explicitly asks to delete that dataset.

## Workflow

1. Distinguish local state changes such as `velen auth import`, `velen auth logout`, or `velen org use` from remote Knowledge Graph memory mutations.
2. For Knowledge Graph memory changes, inspect `velen memory dataset --help` or `velen schema command memory dataset <subcommand> --output json` before guessing flags.
3. Use `velen --org <slug> memory dataset rename <dataset_key> --name <name>` for display-name changes.
4. Use `velen --org <slug> memory dataset delete <dataset_key>` only after the user explicitly asks to remove that dataset.
5. Report the resulting org, dataset key, and Request ID when available.
