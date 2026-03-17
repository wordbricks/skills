# Wordbricks Skills

Installable agent skills published by Wordbricks.

This repo also works as a Claude Code plugin root.

Comment: the Velen CLI now ships its own embedded skill metadata through `velen schema skills`, while this repo packages a separately maintained Claude plugin. Keep the examples here aligned with the live CLI contract rather than assuming the two stay in sync automatically.

## Claude Code Plugin

Load this repo as a local Claude Code plugin:

```bash
claude --plugin-dir /Users/starush/work/skills
```

The plugin namespace is `wordbricks`, so the Velen skill is available as:

```text
/wordbricks:velen
```

## Install

Install the `velen` skill from this repo with either `npx` or `bunx`:

```bash
npx skills add wordbricks/skills --skill velen
```

```bash
bunx skills add wordbricks/skills --skill velen
```

You can also install from a full GitHub URL with either `npx` or `bunx`:

```bash
npx skills add https://github.com/wordbricks/skills --skill velen
```

```bash
bunx skills add https://github.com/wordbricks/skills --skill velen
```

## Available Skills

### `velen`

Use the Velen CLI to authenticate, resolve org context, inspect connected data sources, validate or execute read-only SQL queries, and inspect published insights.

Path: `skills/velen`
