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
- Use `velen schema openapi --output json` when you need the bundled HTTP
  contract. Do not rely on `velen schema skills`; current builds may return an
  empty list because the skill is installed from an external repository.
- Resolve org and source ambiguity before moving on to query execution. Treat
  `flag`, `env`, `workspace`, and `user-default` as distinct org sources.
- Sources used by `source show` and `api` must be provider-qualified, for
  example `postgres://warehouse`. Prefer the same form for `query`; query
  commands can resolve a bare key only when it is unambiguous in the org.
- Workers are separate from sources. Use `velen --org <slug> workers list` for
  connected workers and pass the returned reference, such as
  `hermes://hermes-agent`, directly to `velen api`.

## Workflow

1. For command discovery, start with help or schema commands before auth.
2. For profile discovery, use `velen profile list`; use `--profile <name>` for
   one invocation and `velen profile switch <name>` only when persistence was
   requested.
3. For protected data discovery, confirm auth and org context.
4. Use `velen org current`, `velen org list`, `velen --org <slug> source list`, and `velen --org <slug> source show <provider://source-key>` to narrow a passive source. Use `velen --org <slug> workers list` for an agent or worker instead.
5. When inspecting persisted org selection, report the source and workspace
   config path returned by `velen org current`.
6. Use `velen schema command <path> --output json` when you need the exact current command contract.
7. Run `velen --org <slug> api --source <worker-reference>` without a target to inspect a worker's API descriptor and examples before preparing a request.
