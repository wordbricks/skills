# OneQuery Plugin

This plugin packages the OneQuery Claude Code workflow published from the
Wordbricks marketplace.

## Installation

After adding the marketplace:

```bash
/plugin install onequery@wordbricks
```

## Usage

Invoke the plugin skill with:

```text
/onequery:ask
```

Use it to resolve org and source context, inspect the CLI surface, validate
read-only SQL, and run bounded read-only queries through OneQuery-managed
access.

## Local Development

Load the plugin directly from a checkout of this repository:

```bash
claude --plugin-dir /path/to/skills/plugins/onequery
```

## Team Installation

Add this to your project's `.claude/settings.json` to make the marketplace
available and enable the plugin for the repo:

```json
{
  "extraKnownMarketplaces": {
    "wordbricks": {
      "source": {
        "source": "github",
        "repo": "wordbricks/skills"
      }
    }
  },
  "enabledPlugins": {
    "onequery@wordbricks": true
  }
}
```
