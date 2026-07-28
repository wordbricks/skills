---
name: velen-cli-mutation
description: Use when you need to reason about mutating Velen CLI flows, including source API operations advertised by the current source descriptor, auth/org selection, CLI/skill updates, Knowledge Graph memory writes, or persona memory management.
---

# Velen CLI Mutation

This leaf skill extends the main `SKILL.md` in this directory.

## Guardrails

- Follow the main `SKILL.md` before running local state mutations.
- Inherit its run-context reuse, descriptor, authorization, preview, and
  read-back rules instead of repeating them here.
- Prefer explicit confirmation before changing persistent local CLI state.
- Treat `velen memory dataset delete` as destructive: use it only when the user explicitly asks to delete that dataset.
- Use `velen update --dry-run`, `velen skill update --dry-run`, or
  `velen skill add --dry-run` when the user wants to preview local install
  changes.

## Workflow

1. Distinguish local state changes such as `velen auth import`, `velen auth logout`, or `velen org use` from remote source API or memory mutations.
2. For a source API mutation, use the resolved run context and execute the
   smallest matching operation under the main skill's mutation rules.
3. For Knowledge Graph memory changes, inspect `velen memory dataset --help` or `velen schema command memory dataset <subcommand> --output json` before guessing flags.
4. Use `velen --org <slug> memory dataset describe <dataset_key>` before risky dataset changes when the current scope is unclear.
5. Use `velen --org <slug> memory dataset rename <dataset_key> --name <name>` for display-name changes.
6. Use `velen --org <slug> memory dataset delete <dataset_key>` only after the user explicitly asks to remove that dataset.
7. For explicit persona memory changes, inspect `velen persona --help` or the narrow schema command, then use the smallest matching command: `velen persona profile list`, `velen persona profile upsert`, `velen persona remember`, `velen persona forget`, or `velen persona consolidate`. Use `velen persona forget`, not delete, when the user asks to remove one durable persona memory.
8. For local tool updates, `velen update` updates the binary first and then the packaged `velen-cli` skill; `--package-manager bun|npm` selects the binary installer.
9. Report the resulting org, source and operation or dataset/persona key, local command path, and Request ID when available.
