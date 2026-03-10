---
name: velen-cli
description: Use when the user wants to inspect company or customer data that lives behind Velen, run ad hoc read-only SQL against a Velen-connected source, verify or extend a known Velen insight by public ID, or troubleshoot `velen` auth, org, source, or query failures. Do not use for local databases, direct credentials, or SQL work that bypasses Velen access controls.
---

# Velen CLI

Use `velen` when you need auditable terminal access to a company's Velen-connected data. Prefer it over ad hoc credentials because auth, org access, and source capability checks are enforced by the CLI and server.

## When To Use

- The user wants to check company or customer data that is expected to be accessible through Velen-managed access.
- The request sounds like ad hoc analytics, metric validation, or evidence gathering against a Velen-connected source.
- The user has a known Velen insight public ID and wants to inspect, verify, or extend it.
- The task is to debug `velen` login, org resolution, source lookup, or query execution failures.

## When Not To Use

- The task is about a local database or requires connecting directly with tools or credentials outside Velen instead of going through the CLI.
- The user needs write access, DDL, data mutation, or non-read-only side effects.
- The task is only about Velen product design, API behavior, or documentation and does not require CLI interaction.
- Comment: Provider-specific sources are in scope when Velen is the access path. If the user asks for "warehouse data", "customer metrics", or "run a quick SQL check" without naming Velen, prefer this skill when the expected path is Velen-managed rather than direct credentials.

## Quick Start

1. Confirm the CLI is available with `command -v velen`.
2. If `velen` is missing but you are inside the Velen repo, use `just velen <args>` from `apps/cli`.
3. If the CLI is unavailable outside the repo, stop and ask the user to install it or provide the binary path. Do not invent an install method.
4. Check auth with `velen auth whoami`.
5. If auth fails or the CLI reports that the user is not logged in, run `velen auth login` and complete browser authorization.
6. Resolve org context with `velen org current`. Use `velen org list` and `velen org use <org-slug>` when needed.
7. Inspect sources with `velen source list` and `velen source show <source_key>`.
8. Run read-only SQL with `velen query --source <source_key> --sql "select ..."` or use `--file` or `--stdin` for longer queries.
9. Fetch a known insight with `velen insight get <PUBLIC_ID>`.

## Workflows

### Query Company Data

- Start with `velen source list`.
- Choose a source where `QUERY` is `yes`.
- Run `velen source show <source_key>` to confirm provider, org, status, and the sample query shape.
- Use provider-appropriate read-only SQL only. The CLI rejects writes and other non-read-only statements.
- Prefer small validation queries first: row counts, bounded date ranges, `limit`, and low-cost aggregations.
- For multi-line SQL, prefer `--file <path.sql>` or pipe with `--stdin`.
- If output is truncated, narrow the query and rerun with stronger filters or smaller limits.

### Investigate An Existing Insight

- Use `velen insight get <PUBLIC_ID>` when the user already has the public ID.
- Read the title, timestamps, URL, references, and body.
- Use `velen query` to validate or extend the evidence behind the insight when more analysis is needed.
- Comment: CLI v1 can fetch an insight only by public ID. There is no `insight list` or search command yet, so if the ID is unknown you need the user to provide it or investigate through sources and queries instead.

### Work Across Orgs

- `--org <slug>` overrides the configured active org for a single command.
- `velen org use <slug>` persists the active org for later commands.
- If a command fails because org is unresolved, use `velen org current` and `velen org list` before retrying.

## Failure Recovery

- `Not Logged In`: run `velen auth login`.
- `org_not_found` or `forbidden`: run `velen org list`, then choose an org the user can access.
- `source_not_found`: run `velen source list` and retry with the canonical source key.
- `source_not_queryable`: choose a source where `QUERY` is `yes`.
- `query_rejected`: rewrite the SQL as read-only. Avoid `insert`, `update`, `delete`, DDL, temp table creation, or vendor-specific write side effects.
- `query_execution_unavailable` or `query_execution_timed_out`: retry. The CLI already retries transient failures internally, so repeated failure usually means service instability or an oversized query.
- `failed to read SQL file` or `stdin is required for --stdin`: fix local input handling before retrying.

## Output Cues

- `velen source list` renders `NAME`, `PROVIDER`, `QUERY`, and `STATUS`.
- `velen source show <source_key>` renders provider, org, status, and whether query is supported in CLI v1.
- `velen query ...` prints source, org, row count, elapsed time, then a plain text table.
- `velen insight get ...` prints the insight title, timestamps, URL, references, and body.
- On failures, the CLI prints `Error`, `Command`, `Stage`, `Why`, optional `Request ID`, optional `Hint`, and `Try` suggestions.

## References

- Read `references/command-patterns.md` for concrete command sequences and example analysis flows.
