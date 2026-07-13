---
name: velen-cli-mutation
description: Use when you need to run an explicitly requested Velen CLI mutation such as profile or org changes, source connection, source API or worker actions, CLI/skill updates, Knowledge Graph writes, or persona profile and memory management.
---

# Velen CLI Mutation

This leaf skill extends the main `SKILL.md` in this directory.

## Guardrails

- Follow the applicable routing and authorization guardrails in the main
  `SKILL.md` before running a mutation.
- Keep every local or remote mutation within the exact scope the user
  authorized. Source API operations may mutate connected systems when the
  source descriptor and the user's request permit it.
- Preview body-bearing source API and worker requests with `velen api --dry-run`.
  Inspect each command schema before assuming another mutation supports
  dry-run; `profile switch`, `org create`, `source connect`, Knowledge Graph
  writes, and persona writes currently do not.
- Change persistent local CLI state only when the user asked to persist it.
- Treat `velen memory dataset delete` as destructive: use it only when the user explicitly asks to delete that dataset.
- Use `velen update --dry-run`, `velen skill update --dry-run`, or
  `velen skill add --dry-run` when the user wants to preview local install
  changes.

## Workflow

1. Distinguish local writes (`auth import`, `auth logout`, `profile switch`,
   `org use`, CLI/skill updates) from remote writes (`org create`,
   `source connect`, source API or worker actions, memory, and persona).
2. For a source API mutation, inspect the descriptor, preview the exact method,
   target, headers, and body with `velen api --dry-run`, then execute only the
   authorized operation.
3. For `source connect`, load the provider guide without `--input`, then send
   inline credential JSON only for an explicit connection request. Its
   `--input` does not accept a path or stdin. Build the canonical
   `<provider>://<sourceKey>` reference from the result instead of copying a
   bare `source show` hint.
4. Use `org create` only for an explicit organization creation request. Use
   `org get` to verify the resolved org afterward.
5. For Knowledge Graph memory changes, inspect `velen memory dataset --help` or `velen schema command memory dataset <subcommand> --output json` before guessing flags.
6. Use `velen --org <slug> memory dataset describe <dataset_key>` before risky dataset changes when the current scope is unclear.
7. Use `velen --org <slug> memory dataset rename <dataset_key> --name <name>` for display-name changes.
8. Use `velen --org <slug> memory dataset delete <dataset_key>` only after the user explicitly asks to remove that dataset.
9. For explicit persona changes, inspect the narrow schema command and use the
   smallest matching operation. Use `persona forget` for one durable memory;
   reserve `persona profile delete` for explicit deletion of the whole profile
   and its durable memory. Treat `persona profile publish` as a super-admin
   remote write.
10. For local tool updates, `velen update` updates the binary first and then the packaged `velen-cli` skill; `--package-manager bun|npm` selects the binary installer.
11. For an explicit Hermes Agent request, resolve the reference with
   `velen --org <slug> workers list`, inspect the descriptor with
   `velen --org <slug> api --source hermes://<worker-key>`, preview the exact
   `/v1/responses` or `/v1/runs` POST, then execute it. Never route this flow
   through the Chronos-only `/api/cron/fire` webhook.
12. Report the resulting org, source or worker reference, dataset key or persona key,
   local command path, and Request ID when available.
