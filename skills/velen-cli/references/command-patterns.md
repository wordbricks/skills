# Command Patterns

Use these sequences when the user asks to query company data, inspect a source, or investigate a known insight.

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

## Inspect Sources Before Querying

```bash
velen source list
velen source show warehouse
```

Use the source list to pick a source where `QUERY` is `yes`. Use `source show` to confirm the canonical `source_key`, provider, and org before writing SQL.

## Run A Short Validation Query

```bash
velen query --source warehouse --sql "select 1"
```

Use this first when you want to verify auth, org, source selection, and queryability without spending time on a large query.

## Run A Real Analysis Query

```bash
velen query --source warehouse --sql "select date_trunc('day', created_at) as day, count(*) as signups from users where created_at >= current_date - interval '7 days' group by 1 order by 1 desc"
```

Guidance:

- Add explicit date bounds.
- Add `limit` where it makes sense.
- Start with aggregate checks before wide row dumps or heavy joins.
- Tailor SQL dialect to the provider shown by `velen source show`.

## Use File Or Stdin For Longer SQL

```bash
velen query --source warehouse --file ./analysis.sql
cat ./analysis.sql | velen query --source warehouse --stdin
velen --org acme query --source warehouse --file ./analysis.sql
```

Prefer `--file` or `--stdin` for multi-line SQL so the query can be inspected, revised, and rerun cleanly.

## Investigate A Known Insight

```bash
velen insight get ACME-13
```

Use the returned references and body to decide whether you need follow-up queries. If the public ID is unknown, the CLI cannot list insights yet, so you need the user to provide the ID or you need to investigate through sources and queries instead.

## Common Recovery Moves

```bash
velen auth login
velen org list
velen source list
velen source show warehouse
```

Map failures to the smallest recovery step first:

- Auth problems: `velen auth login`
- Org problems: `velen org list`
- Source lookup problems: `velen source list`
- Queryability problems: pick a source with `QUERY` set to `yes`
