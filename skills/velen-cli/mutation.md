---
name: velen-cli-mutation
description: Use when you need to reason about future mutating Velen CLI flows or local state changes such as auth/org selection without assuming data-plane write access.
---

# Velen CLI Mutation

This leaf skill extends the main `SKILL.md` in this directory.

## Guardrails

- Follow the main `SKILL.md` before running local state mutations.
- Do not assume write access to remote data-plane resources through the public CLI surface.
- Prefer explicit confirmation before changing persistent local CLI state.

## Workflow

1. Distinguish local state changes such as `velen auth import`, `velen auth logout`, or `velen org use` from remote data mutations.
2. Confirm why the state change is needed before applying it.
3. Report the resulting local state clearly after the command completes.
