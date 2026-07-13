---
name: velen-cli-mutation
description: Use when you need to reason about mutating Velen CLI flows or local state changes such as auth/org selection, CLI/skill updates, Knowledge Graph memory writes, or persona memory management without assuming general data-plane write access.
---

# Velen CLI Mutation

This leaf skill extends the main `SKILL.md` in this directory.

## Guardrails

- Follow the main `SKILL.md` before running local state mutations.
- Do not assume write access to remote data-plane resources through the public
  CLI surface except for explicit user-requested Knowledge Graph memory,
  persona memory, or connected-worker actions.
- Treat a connected-worker request as remote work. Send one only when the user
  explicitly asks for that action, preview body-bearing requests with
  `velen api --dry-run`, and keep the request within the named worker and task.
- Prefer explicit confirmation before changing persistent local CLI state.
- Treat `velen memory dataset delete` as destructive: use it only when the user explicitly asks to delete that dataset.
- Use `velen update --dry-run`, `velen skill update --dry-run`, or
  `velen skill add --dry-run` when the user wants to preview local install
  changes.

## Workflow

1. Distinguish local state changes such as `velen auth import`, `velen auth logout`, or `velen org use` from remote Knowledge Graph memory mutations.
2. For Knowledge Graph memory changes, inspect `velen memory dataset --help` or `velen schema command memory dataset <subcommand> --output json` before guessing flags.
3. Use `velen --org <slug> memory dataset describe <dataset_key>` before risky dataset changes when the current scope is unclear.
4. Use `velen --org <slug> memory dataset rename <dataset_key> --name <name>` for display-name changes.
5. Use `velen --org <slug> memory dataset delete <dataset_key>` only after the user explicitly asks to remove that dataset.
6. For explicit persona memory changes, inspect `velen persona --help` or the narrow schema command, then use the smallest matching command: `velen persona profile list`, `velen persona profile upsert`, `velen persona remember`, `velen persona forget`, or `velen persona consolidate`. Use `velen persona forget`, not delete, when the user asks to remove one durable persona memory.
7. For local tool updates, `velen update` updates the binary first and then the packaged `velen-cli` skill; `--package-manager bun|npm` selects the binary installer.
8. For an explicit Hermes Agent request, resolve the reference with
   `velen --org <slug> workers list`, inspect the descriptor with
   `velen --org <slug> api --source hermes://<worker-key>`, preview the exact
   `/v1/responses` or `/v1/runs` POST, then execute it. Never route this flow
   through the Chronos-only `/api/cron/fire` webhook.
9. Report the resulting org, worker reference, dataset key or persona key,
   local command path, and Request ID when available.
