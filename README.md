# Wordbricks Plugins And Skills

This repository contains the Wordbricks Claude Code marketplace plugins plus standalone skills.

## Available Plugins

- [`onequery`](./plugins/onequery/README.md): Use the OneQuery CLI from Claude
  Code for authenticated org discovery, source inspection, and read-only query
  workflows.
- [`velen`](./plugins/velen/README.md): Use the Velen CLI from Claude Code for
  authenticated org discovery, insight inspection, and read-only Velen-managed
  data access.

## Available Skills

- [`onequery-cli`](./skills/onequery-cli/SKILL.md)
- [`velen-cli`](./skills/velen-cli/SKILL.md)

## Installation

### Add the marketplace

Add this marketplace to Claude Code:

```bash
/plugin marketplace add wordbricks/skills
```

### Browse or install a plugin

After adding the marketplace:

```bash
/plugin
/plugin install <plugin>@wordbricks
```

See each plugin README for exact install commands, usage, local development,
and team configuration:

- [`plugins/onequery/README.md`](./plugins/onequery/README.md)
- [`plugins/velen/README.md`](./plugins/velen/README.md)

### Team installation

For team-wide marketplace availability, add this to your project's
`.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "wordbricks": {
      "source": {
        "source": "github",
        "repo": "wordbricks/skills"
      }
    }
  }
}
```

Then enable the specific plugin entries you want from the matching plugin
README.
