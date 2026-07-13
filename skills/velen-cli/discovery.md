---
name: velen-cli-discovery
description: Use when you need to inspect org context, available sources, or the CLI's own machine-readable schema surface before issuing a narrower Velen command.
---

# Velen CLI Discovery

This leaf skill extends the main `SKILL.md` in this directory.

## Guardrails

- Follow the main `SKILL.md` before running discovery commands.
- Command-only discovery can run before auth.
- Use `velen --help` and `velen <command> --help` for human-readable command discovery.
- Prefer `velen schema commands --output json` when you need command capabilities.
- Use `velen schema skills --output json` when you need packaged skill metadata.
- Resolve org and source ambiguity before moving on to query execution. Treat
  `flag`, `env`, `workspace`, and `user-default` as distinct org sources.
- Sources used by `source show`, `query`, and `api` must be
  provider-qualified, for example `postgres://warehouse`.
- Workers are separate from sources. Use `velen --org <slug> workers list` for
  connected workers and pass the returned reference, such as
  `hermes://hermes-agent`, directly to `velen api`.

## Workflow

1. For command discovery, start with help or schema commands before auth.
2. For protected data discovery, confirm auth and org context.
3. Use `velen org current`, `velen org list`, `velen --org <slug> source list`, and `velen --org <slug> source show <provider://source-key>` to narrow a passive source. Use `velen --org <slug> workers list` for an agent or worker instead.
4. When inspecting persisted org selection, report the source and workspace
   config path returned by `velen org current`.
5. Use `velen schema command <path> --output json` when you need the exact current command contract.
6. Run `velen --org <slug> api --source <worker-reference>` without a target to inspect a worker's API descriptor and examples before preparing a request.
