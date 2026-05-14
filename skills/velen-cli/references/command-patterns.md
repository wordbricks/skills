# Command Patterns

Use these sequences when the user asks to query company data, inspect a
source, investigate a known insight, or discover the Velen CLI surface.

## Discover The CLI Surface

```bash
velen --help
velen org --help
velen source --help
velen query execute --help
velen query validate --help
velen memory --help
velen memory dataset --help
velen memory remember --help
velen memory recall --help
velen schema commands --output json
velen schema command query execute --output json
velen schema command memory recall --output json
```

Use these when command shape, read controls, or packaged guardrails are
unclear. Start with top-level help or `schema commands`, then inspect the
narrowest subcommand before guessing flags.

## Establish Context

```bash
command -v velen
velen auth whoami
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
velen --org acme source show warehouse
```

Use the source list to pick a source where `QUERY` is `yes`. Use `source show`
to confirm the canonical `source_key`, provider, status, org, and query support
before writing SQL.

## Validate Query Shape Before Execution

```bash
velen --org acme query validate --source warehouse --sql "select 1"
```

Use `query validate` first when the SQL is unfamiliar, the provider dialect is
uncertain, or you want a cheap syntax and access check before execution.

## Run A Short Validation Query

```bash
velen --org acme query execute --source warehouse --sql "select 1" --max-rows 10
```

Use this first when you want to verify auth, org, source selection, and queryability without spending time on a large query.

## Run A Real Analysis Query

```bash
velen --org acme query execute --source warehouse --sql "select date_trunc('day', created_at) as day, count(*) as signups from users where created_at >= current_date - interval '7 days' group by 1 order by 1 desc" --max-rows 200 --max-bytes 50000 --cell-max-chars 500
```

Guidance:

- Add explicit date bounds.
- Add `limit` where it makes sense.
- Start with aggregate checks before wide row dumps or heavy joins.
- Tailor SQL dialect to the provider shown by
  `velen --org <slug> source show`.
- Use `--page-size`, `--max-rows`, `--max-bytes`, and `--cell-max-chars` when
  tighter read bounds materially reduce payload size.
- Use `--request-id <id>` when you want stable trace correlation across
  multiple related CLI calls.

## Use File Or Stdin For Longer SQL

```bash
velen --org acme query validate --source warehouse --file ./analysis.sql
velen --org acme query execute --source warehouse --file ./analysis.sql --max-rows 200
cat ./analysis.sql | velen --org acme query execute --source warehouse --stdin
```

Prefer `--file` or `--stdin` for multi-line SQL so the query can be inspected, revised, and rerun cleanly.

## Query Company Data

- Resolve the org first, then inspect sources in that org.
- Narrow source selection by the most relevant product name or provider before inspecting multiple candidates.
- Confirm the source with `velen --org <slug> source show <source_key>`.
- Start with a bounded aggregate or `select 1` before wider inspection.

## Validate A Metric

- Start with the smallest aggregate or count that tests the metric definition.
- Add explicit date bounds before expanding the query.
- Prefer follow-up breakdowns only after the base aggregate looks correct.

## Work Across Orgs

```bash
velen --org acme source list
velen --org acme query execute --source warehouse --sql "select 1"
velen org use acme
```

Use `--org <slug>` for one-off checks. Use `velen org use <slug>` only when the rest of the session should stay pinned to that org.

## Resolve Informal Source Names

- If the user gives a product name, environment, or nickname instead of a canonical source key, treat it as an alias to resolve.
- Prefer exact source-key or source-name matches first, then obvious prefix matches.
- If multiple queryable sources still fit, report the ambiguity before querying.

## Investigate A Known Insight

```bash
velen --org acme insight list
velen --org acme insight get ACME-13
```

Use the returned references and body to decide whether you need follow-up
queries. If the public ID is unknown, use `insight list` in the target org to
discover visible insights before falling back to source queries.

## Manage Knowledge Graph Memory

```bash
velen --org acme memory status
velen --org acme memory dataset list
velen --org acme memory dataset create warehouse --name "Warehouse Knowledge" --description "Tables, fields, joins, metric definitions, and analysis caveats for warehouse questions"
```

Use focused dataset keys such as `warehouse`, `metrics`, `customers`,
`business-rules`, or `incidents`. Prefer `--org <slug>` instead of relying on
persisted org state.

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

## Common Recovery Moves

```bash
velen auth login
velen auth import --input ./session.json
velen org list
velen --org acme source list
velen --org acme source show warehouse
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
