# Velen Plugin

This plugin packages the Velen Claude Code workflow published from the
Wordbricks marketplace.

## Installation

After adding the marketplace:

```bash
/plugin install velen@wordbricks
```

## Usage

Invoke the plugin skill with:

```text
/velen:ask
```

Use it to authenticate with the Velen CLI, resolve org and source context,
inspect published insights, and run bounded read-only queries through
Velen-managed access.

## Local Development

Load the plugin directly from a checkout of this repository:

```bash
claude --plugin-dir /path/to/skills/plugins/velen
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
    "velen@wordbricks": true
  }
}
```
