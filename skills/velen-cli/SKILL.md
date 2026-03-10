---
name: velen-cli
description: Use when the user wants to inspect company or customer data that lives behind Velen, run ad hoc read-only SQL against a Velen-connected source, or verify or extend a known Velen insight by public ID. Do not use for local databases, direct credentials, or SQL work that bypasses Velen access controls.
---

# Velen CLI

Use `velen` when you need auditable terminal access to a company's Velen-connected data. Prefer it over ad hoc credentials because auth, org access, and source capability checks are enforced by the CLI and server.

## Overview

This skill is for read-only analysis through Velen-managed access.

- Use it when the user wants company or customer data that is expected to be available through Velen, wants to validate a metric with ad hoc SQL, or already has a Velen insight public ID to inspect.
- Do not use it for local databases, direct credentials, write operations, DDL, or product/documentation questions that do not require CLI access.
- Comment: Provider-specific sources are still in scope when Velen is the access path. If the user asks for "warehouse data", "customer metrics", or "run a quick SQL check" without naming Velen, prefer this skill when the expected path is Velen-managed rather than direct credentials.

## Prerequisites

- `velen` must be installed. If it is missing, install it with `npm install -g @wordbricks/velen`.
- The user must be able to complete browser login if `velen auth login` is required.
- The task must stay read-only.

## Required Workflow

**Follow these steps in order. Do not skip steps.**

### Step 1: Confirm CLI availability and auth

1. Run `command -v velen`.
2. If `velen` is missing, install it with `npm install -g @wordbricks/velen`.
3. Run `velen auth whoami`.
4. If auth is missing or expired, run `velen auth login` and complete browser authorization.

### Step 2: Resolve org context

1. Run `velen org current`.
2. If the active org is unclear or wrong, run `velen org list`.
3. Use `velen org use <slug>` to persist the org for later commands, or `--org <slug>` for a single command.

### Step 3: Choose the right source or insight entry point

1. If the user already has an insight public ID, start with `velen insight get <PUBLIC_ID>`.
2. Otherwise run `velen source list` and choose a source where `QUERY` is `yes`.
3. Run `velen source show <source_key>` to confirm provider, org, status, and query support before writing SQL.

### Step 4: Run the smallest useful read-only operation

1. For SQL work, start with a cheap validation query such as `select 1`, a row count, or a bounded aggregate.
2. Use provider-appropriate read-only SQL only.
3. Prefer `--file <path.sql>` or `--stdin` for multi-line SQL.
4. If output is truncated or too broad, narrow the query and rerun with stronger filters, bounded dates, or smaller limits.

### Step 5: Summarize evidence and next action

1. Report the org, source, and query or insight used.
2. Call out any ambiguity in source choice, org context, or missing insight ID.
3. If more evidence is needed, propose the next smallest follow-up query.

## Practical Workflows

- Query company data: Use `velen source list`, inspect the chosen source, then run a bounded read-only query.
- Validate a metric: Start with a small aggregate or count before wider breakdowns or row-level inspection.
- Investigate an existing insight: Use `velen insight get <PUBLIC_ID>`, then extend the evidence with `velen query` only if the returned body and references are insufficient.
- Work across orgs: Use `--org <slug>` for one-off commands and `velen org use <slug>` when the rest of the session should stay in that org.
- Comment: CLI v1 can fetch an insight only by public ID. There is no `insight list` or search command yet, so if the ID is unknown you need the user to provide it or investigate through sources and queries instead.

## Output

- `velen source list` renders `NAME`, `PROVIDER`, `QUERY`, and `STATUS`.
- `velen source show <source_key>` renders provider, org, status, and whether query is supported in CLI v1.
- `velen query ...` prints source, org, row count, elapsed time, then a plain text table.
- `velen insight get ...` prints the insight title, timestamps, URL, references, and body.
- On failures, the CLI prints `Error`, `Command`, `Stage`, `Why`, optional `Request ID`, optional `Hint`, and `Try` suggestions.

## References

- Read `references/command-patterns.md` for concrete command sequences and example analysis flows.
