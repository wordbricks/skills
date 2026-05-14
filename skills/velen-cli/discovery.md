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
- Resolve org and source ambiguity before moving on to query execution.

## Workflow

1. For command discovery, start with help or schema commands before auth.
2. For protected data discovery, confirm auth and org context.
3. Use `velen org current`, `velen org list`, `velen --org <slug> source list`, and `velen --org <slug> source show` to narrow the target.
4. Use `velen schema command <path> --output json` when you need the exact current command contract.
