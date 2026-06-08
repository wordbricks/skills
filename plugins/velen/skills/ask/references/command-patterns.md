# Command Patterns

Use these sequences when the user asks to query company data, inspect a
source, explicitly investigate a past insight, or discover the Velen CLI
surface.

## Discover The CLI Surface

```bash
velen --help
velen org --help
velen source --help
velen source connect --help
velen api --help
velen use --help
velen persona --help
velen persona chat --help
velen persona profile --help
velen persona profile list --help
velen persona profile public list --help
velen persona profile copy --help
velen persona remember --help
velen persona consolidate --help
velen query execute --help
velen query validate --help
velen update --help
velen skill update --help
velen memory --help
velen memory dataset --help
velen memory dataset describe --help
velen memory dataset rename --help
velen memory dataset delete --help
velen memory remember --help
velen memory recall --help
velen schema commands --output json
velen schema skills --output json
velen schema command query execute --output json
velen schema command update --output json
velen schema command persona chat --output json
velen schema command persona profile list --output json
velen schema command persona profile public list --output json
velen schema command persona profile copy --output json
velen schema command persona remember --output json
velen schema command persona consolidate --output json
velen schema command memory dataset rename --output json
velen schema command memory dataset delete --output json
velen schema command memory recall --output json
```

Use these when command shape, read controls, or packaged guardrails are
unclear. Start with top-level help or `schema commands`, then inspect the
narrowest subcommand before guessing flags.

## Establish Context

```bash
command -v velen
velen auth whoami
velen profile list
velen org current
velen org list
velen org use acme
```

Interpretation:

- `velen auth whoami` confirms the current identity and the effective org source.
- `velen org current` shows whether org resolution came from `--org`, config, or is unresolved.
- `velen org list` is the recovery path when org access or org selection is unclear.
- Prefer `velen --org <slug> ...` once the org is known instead of mutating
  persisted local state unless the user wants that change.

## Inspect Sources Before Querying

```bash
velen --org acme source list
velen --org acme source show postgres://warehouse
```

Use the source list to pick a source where `QUERY` is `yes` for SQL work, or
the matching provider/source reference for source API work. Use `source show`
to confirm the provider-qualified source reference, provider, status, org, query
support, and sample query or source identity before writing SQL or calling
`velen api`. Pass that
`<provider>://<source-key>` reference to `source show`, `query`, and `api`
commands.

## Validate Query Shape Before Execution

```bash
velen --org acme query validate --source postgres://warehouse --sql "select 1"
```

Use `query validate` first when the SQL is unfamiliar, the provider dialect is
uncertain, or you want a cheap syntax and access check before execution.

## Run A Short Validation Query

```bash
velen --org acme query execute --source postgres://warehouse --sql "select 1" --max-rows 10
```

Use this first when you want to verify auth, org, source selection, and queryability without spending time on a large query.

## Run A Real Analysis Query

```bash
velen --org acme query execute --source postgres://warehouse --sql "select date_trunc('day', created_at) as day, count(*) as signups from users where created_at >= current_date - interval '7 days' group by 1 order by 1 desc" --max-rows 200 --max-bytes 50000 --cell-max-chars 500
```

Guidance:

- Add explicit date bounds.
- Add `limit` where it makes sense.
- Start with aggregate checks before wide row dumps or heavy joins.
- Tailor SQL dialect to the provider shown by
  `velen --org <slug> source show <provider://source-key>`.
- Use `--page-size`, `--max-rows`, `--max-bytes`, and `--cell-max-chars` when
  tighter read bounds materially reduce payload size.
- Use `--request-id <id>` when you want stable trace correlation across
  multiple related CLI calls.

## Use File Or Stdin For Longer SQL

```bash
velen --org acme query validate --source postgres://warehouse --file ./analysis.sql
velen --org acme query execute --source postgres://warehouse --file ./analysis.sql --max-rows 200
cat ./analysis.sql | velen --org acme query execute --source postgres://warehouse --stdin
```

Prefer `--file` or `--stdin` for multi-line SQL so the query can be inspected, revised, and rerun cleanly.

## Use Raw Query Payload Input

```bash
cat ./query-request.json | velen --org acme query execute --source postgres://warehouse --input -
velen --org acme query validate --source postgres://warehouse --input ./query-request.json
```

Use `--input <path|->` when you need to send the full JSON query request body,
including parameters or request fields that are awkward as convenience flags.
Do not combine `--input` with `--sql`, `--file`, `--stdin`, `--max-rows`,
`--max-bytes`, `--cell-max-chars`, or `--timeout-ms`.

## Use A Non-SQL Source API

```bash
velen use --source slack
velen --org acme api --source slack://workspace --dry-run
velen --org acme api --source slack://workspace --op list_channels --paginate --max-pages 2 --output json
```

Use `velen use --source <provider>` to inspect provider-specific SDK guidance
for non-SQL sources. Use `velen api` only through the Velen-managed source
reference. Start with `--dry-run` when target inference, operation name,
pagination, headers, method, or request body shape is uncertain. Keep operations
read-only. Use `--input <JSON|PATH|->` for request bodies and
`--paginate --max-pages <n>` for bounded pagination.

## Query Company Data

- Resolve the org first, then inspect sources in that org.
- Narrow source selection by the most relevant product name or provider before inspecting multiple candidates.
- Confirm the source with `velen --org <slug> source show <provider://source-key>`.
- For SQL, start with a bounded aggregate or `select 1` before wider inspection.
- For source API, start with `velen api --dry-run` before executing a paginated
  or body-bearing request.

## Validate A Metric

- Start with the smallest aggregate or count that tests the metric definition.
- Add explicit date bounds before expanding the query.
- Prefer follow-up breakdowns only after the base aggregate looks correct.

## Analyze Product KPIs And Funnels

- Translate the user's named metric into the behavior the business actually
  wants before querying deeply. A button click can be raw intent, while a
  request-sent or success event may be the meaningful attempt.
- Verify event semantics from event properties, source metadata, Knowledge Graph
  memory, and local code when available. Locate where the event fires and what
  can happen before or after it.
- Map the minimum funnel needed for the decision, for example:
  `view -> click -> auth gate -> validation/upload -> request sent -> success/error`.
- If the current events cannot distinguish proxy intent from downstream value,
  recommend the smallest additive instrumentation fix as an action item before
  product optimization ideas. Example: keep `generate_click`, add
  `generate_request_sent`, and use `generate_success / generate_request_sent`
  as a guardrail.
- Report primary KPI, secondary KPI, and guardrail separately so raw event
  growth cannot be mistaken for actual improvement.

## Work Across Orgs

```bash
velen --org acme source list
velen --org acme query execute --source postgres://warehouse --sql "select 1"
velen org use acme
```

Use `--org <slug>` for one-off checks. Use `velen org use <slug>` only when the rest of the session should stay pinned to that org.

## Resolve Informal Source Names

- If the user gives a product name, environment, or nickname instead of a canonical provider-qualified source reference, treat it as an alias to resolve.
- Prefer exact source-reference, source-key, or source-name matches first, then obvious prefix matches.
- If multiple queryable sources still fit, report the ambiguity before querying.

## Investigate An Explicitly Requested Past Insight

```bash
velen --org acme insight list
velen --org acme insight get ACME-13
```

Only use this pattern when the user directly asks to inspect past insights,
provides a public insight ID, or asks to verify or extend a specific existing
insight. Do not use `insight list` or `insight get` as a default discovery step
for new analysis. Use the returned references and body to decide whether you
need follow-up queries. If the user explicitly asks for past insight lookup but
the public ID is unknown, use `insight list` in the target org to discover
visible insights before falling back to source queries.

## Manage Knowledge Graph Memory

```bash
velen --org acme memory status
velen --org acme memory dataset list
velen --org acme memory dataset create warehouse --name "Warehouse Knowledge" --description "Tables, fields, joins, metric definitions, and analysis caveats for warehouse questions"
velen --org acme memory dataset describe warehouse
velen --org acme memory dataset rename warehouse --name "Warehouse Knowledge Base"
velen --org acme memory dataset delete stale-notes
```

Use focused dataset keys such as `warehouse`, `metrics`, `customers`,
`business-rules`, or `incidents`. Prefer `--org <slug>` instead of relying on
persisted org state.

Use `memory dataset rename` for human-readable label changes when the dataset
key and scope are still correct. Use `memory dataset delete` only when the user
explicitly asks to remove that org-scoped Knowledge Graph dataset.

## Remember Curated Knowledge

```bash
velen --org acme memory remember --dataset warehouse --file ./knowledge/warehouse-metrics.md --file-name warehouse-metrics.md
velen --org acme memory remember --dataset warehouse --text "orders.created_at is UTC. Revenue uses orders.net_amount, not orders.gross_amount." --file-name warehouse-revenue-rule.md
```

Guidance:

- Store reviewed facts, schema notes, metric definitions, caveats, and source
  provenance.
- Do not store raw broad query output, secrets, credentials, or unreviewed
  customer-sensitive dumps.
- Use stable `--file-name` values so later operators can identify the memory
  item.
- Keep each memory item narrow enough that recall can return a useful context
  block.

## Verify Knowledge Graph Recall

```bash
velen --org acme memory recall --dataset warehouse --query "How should revenue be calculated from orders?" --top-k 5 --output json
```

Run recall before and after writes when improving an existing dataset. If recall
returns irrelevant context, add narrower facts, clearer aliases, or a smaller
dataset rather than dumping more content into `manual`.

## Upsert Structured Graph Memory

First verify the command exists:

```bash
velen memory graph upsert --help
velen schema command memory graph upsert --output json
```

If available, use stable canonical node ids and explicit edges:

```json
{
  "provenance": {
    "sourceKey": "warehouse",
    "sourceKind": "postgres",
    "description": "Warehouse schema and metric rules"
  },
  "nodes": [
    {
      "id": "source:warehouse",
      "kind": "DataSource",
      "name": "warehouse"
    },
    {
      "id": "asset:warehouse.public.orders",
      "kind": "DataAsset",
      "name": "orders"
    },
    {
      "id": "field:warehouse.public.orders.net_amount",
      "kind": "Field",
      "name": "net_amount",
      "description": "Preferred field for revenue calculations"
    },
    {
      "id": "metric:revenue",
      "kind": "Metric",
      "name": "Revenue"
    }
  ],
  "edges": [
    {
      "from": "field:warehouse.public.orders.net_amount",
      "type": "belongs_to",
      "to": "asset:warehouse.public.orders"
    },
    {
      "from": "metric:revenue",
      "type": "derived_from",
      "to": "field:warehouse.public.orders.net_amount"
    }
  ]
}
```

```bash
velen --org acme memory graph upsert --dataset warehouse --file ./knowledge/warehouse.graph.json --file-name warehouse.graph.md
```

Use `DataSource`, `DataAsset`, `Field`, `Metric`, `BusinessConcept`, and
`AnalysisRule` as common node kinds. Use relation types such as `belongs_to`,
`derived_from`, `applies_to`, `warns_against`, and `recommends`.

## Manage Persona Memory

```bash
velen --org acme persona profile list
velen --org acme persona profile public list
velen --org acme persona profile copy sophia
velen --org acme persona profile upsert sophia --file ./personas/sophia.profile.json --display-name "Sophia" --persona-version 2026-06
velen --org acme persona remember sophia --kind style --title "Review tone" --summary "Prefer direct critique with concrete evidence and caveats." --confidence 0.9 --privacy internal
velen --org acme persona consolidate sophia
```

Use persona commands only when the user explicitly asks to inspect or manage
DB-backed persona profiles or durable persona memory. Use `--user-scoped` when
the requested memory should apply only to the current CLI user rather than the
org-wide persona.

## Update The CLI Or Agent Skill

```bash
velen update --dry-run
velen update
velen update --package-manager npm
velen skill update --dry-run
velen skill update
```

`velen update` updates the globally installed `@wordbricks/velen` binary first
and then installs or updates the packaged `velen-cli` agent skill from
`wordbricks/skills`. Use `skill update` when only the skill should be refreshed.

## Common Recovery Moves

```bash
velen auth login
velen auth import --input ./session.json
velen org list
velen --org acme source list
velen --org acme source show postgres://warehouse
velen --org acme memory status
velen --org acme memory dataset list
velen query execute --help
```

Map failures to the smallest recovery step first:

- Auth problems: `velen auth login` or `velen auth import --input ...`
- Org problems: `velen org list`
- Source lookup problems: `velen --org <slug> source list`
- Query shape problems: `velen query execute --help` or
  `velen schema command query execute --output json`
- Queryability problems: pick a source with `QUERY` set to `yes`
- Knowledge Graph memory problems: `velen --org <slug> memory status`,
  `velen --org <slug> memory dataset list`, then retry the smallest
  `memory remember` or `memory recall` command
