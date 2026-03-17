# Wordbricks Plugins For Claude Code

This repository is a Claude Code marketplace containing the `velen` plugin.

Comment: the Velen CLI now ships its own embedded skill metadata through `velen schema skills`, while this repo packages a separately maintained Claude plugin. Keep the examples here aligned with the live CLI contract rather than assuming the two stay in sync automatically.

## Installation

### Add the marketplace

Add this marketplace to Claude Code:

```bash
/plugin marketplace add wordbricks/skills
```

### Install the `velen` plugin

After adding the marketplace:

```bash
/plugin install velen@wordbricks
```

Or browse interactively:

```bash
/plugin
```

### Local development

Load the plugin directly from this checkout:

```bash
claude --plugin-dir /Users/starush/work/skills/plugins/velen
```

## Usage

Invoke the plugin skill with:

```text
/velen:ask
```

Use it to authenticate with the Velen CLI, resolve org and source context, inspect published insights, and run bounded read-only queries through Velen-managed access.

### Team installation

For team-wide plugin usage, add this to your project's `.claude/settings.json`:

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

Team members will be prompted to install the plugin when they trust the repository.

## Structure

- Marketplace metadata: `.claude-plugin/marketplace.json`
- Plugin root: `plugins/velen`
- Skill path: `plugins/velen/skills/ask`
