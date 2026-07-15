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
- Resolve org and source ambiguity before moving on to query execution.
- Sources used by `source show`, `query`, and `api` must be
  provider-qualified, for example `postgres://warehouse`.
- Use `velen --org <slug> api --source <provider://source-key> --output json`
  without an operation or target to discover the current source API descriptor.
- Treat that descriptor as the source of truth for provider operations, input
  policy, selectors, pagination, examples, and notes. Do not infer capabilities
  from the provider name, skill text, or an earlier run.

## Workflow

1. For command discovery, start with help or schema commands before auth.
2. For protected data discovery, confirm auth and org context.
3. Use `velen org current`, `velen org list`, `velen --org <slug> source list`, and `velen --org <slug> source show <provider://source-key>` to narrow the target.
4. For source API work, inspect the selected source's descriptor and report the
   relevant operation contract before preparing a request.
5. Use `velen schema command <path> --output json` when you need the exact current command contract.
