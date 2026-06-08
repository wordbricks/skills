---
name: velen-cli-query
description: Use when you need to run a read-only SQL query against a Velen-managed source after org and source selection are already clear.
---

# Velen CLI Query

This leaf skill extends the main `SKILL.md` in this directory.

## Guardrails

- Follow the main `SKILL.md` before running query commands.
- Use `velen query validate` before `velen query execute` when SQL shape, access, or provider dialect is uncertain.
- Source arguments must be provider-qualified references such as
  `postgres://warehouse`; resolve them through `velen source list` or
  `velen source show`.
- Start execution with the smallest cheap query that can answer the question.
- For KPI, funnel, product analytics, or conversion questions, verify event
  semantics before optimizing on an event count. Check whether the event is raw
  intent, request sent, success, or downstream value, and call out missing
  instrumentation as an action item when needed.
- Prefer file or stdin input for multi-line SQL to avoid shell escaping mistakes.
- Use `--max-rows`, `--max-bytes`, `--cell-max-chars`, and SQL filters to bound result size before widening reads.
- Use `--input <path|->` only when sending a full JSON query request payload;
  do not combine it with SQL convenience flags or result-window flags.
- Prefer the CLI's built-in 180-second request timeout. Omit global
  `--timeout` for normal query work; use query `--timeout-ms <ms>` for a single
  slow query result window, or global `--timeout <sec>` only as an explicit
  invocation-specific override instead of editing persisted config.

## Workflow

1. Confirm the source is queryable with `velen --org <slug> source show <provider://source-key>`.
2. Validate unfamiliar SQL with `velen --org <slug> query validate --source <provider://source-key> ...`.
3. For KPI/product analytics, map the event funnel and identify whether a new
   event or property is required before treating the queried metric as primary.
4. Execute a bounded read-only SQL statement with `velen --org <slug> query execute --source <provider://source-key> ...`.
5. Tighten filters or reduce result width before widening scope.
