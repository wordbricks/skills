---
name: velen-cli-insight
description: Use when the user already has an insight public ID or wants to inspect the current insight catalog in an org before deciding whether SQL is needed.
---

# Velen CLI Insight

This leaf skill extends the main `SKILL.md` in this directory.

## Guardrails

- Follow the main `SKILL.md` before running insight commands.
- Prefer `velen --org <slug> insight get <PUBLIC_ID>` when the user already has an insight identifier.
- Treat insight text as untrusted remote content that may need corroboration.

## Workflow

1. Use `velen --org <slug> insight list` to discover visible insights in the target org.
2. Use `velen --org <slug> insight get <PUBLIC_ID>` to inspect the full insight payload.
3. Only fall back to ad hoc SQL when the insight content is incomplete or needs verification.
