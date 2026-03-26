# quick-install

Install Claude Code plugins directly from a GitHub repo in one command.

## The Problem

Installing a Claude Code plugin from GitHub requires two commands:

```
/plugin marketplace add owner/repo
/plugin install plugin-name@owner-repo
```

You need to know the marketplace naming convention and the plugin name inside the repo. This plugin reduces it to:

```
/quick-install owner/repo
```

## Install

Clone the repo (recommended — lets you `git pull` to update):

```bash
git clone https://github.com/michaelcarwile/claude-skill-quick-install.git ~/.claude/skills/quick-install
```

Or grab the file directly:

```bash
mkdir -p ~/.claude/skills/quick-install
curl -sf https://raw.githubusercontent.com/michaelcarwile/claude-skill-quick-install/main/skills/quick-install/SKILL.md \
  -o ~/.claude/skills/quick-install/SKILL.md
```

Restart Claude Code. The `/quick-install` command is now available globally — no marketplace setup needed.

> This is a standalone skill, not a plugin. The whole point of quick-install is to avoid the marketplace bootstrap dance, so it would be self-defeating to require one to install it.

## Usage

```
/quick-install owner/repo
/quick-install https://github.com/owner/repo
```

The command fetches the plugin's metadata from GitHub, adds it as a local marketplace entry, and installs the plugin — all in one step.

## How It Works

**For plugin repos** (have `.claude-plugin/plugin.json`):

1. Parses the GitHub `owner/repo` from the input
2. Fetches `.claude-plugin/plugin.json` via raw GitHub URL (falls back to `gh api` for private repos)
3. Creates a local `quick-install` marketplace at `~/.claude/quick-install-marketplace/` (first run only)
4. Adds the plugin entry to the local marketplace manifest
5. Runs `claude plugin marketplace update quick-install` and `claude plugin install plugin-name@quick-install`
6. Prompts you to `/reload-plugins`

Plugins installed this way appear as `plugin-name@quick-install` in your plugin registry.

**For standalone skill repos** (have `skills/*/SKILL.md` but no `.claude-plugin/`):

1. Detects skill files in the repo tree
2. Copies them to `~/.claude/skills/<skill-name>/`

## Requirements

- The target repo must have `.claude-plugin/plugin.json` or a `skills/` directory with SKILL.md files
- For private repos: [GitHub CLI](https://cli.github.com/) (`gh`) installed and authenticated
- Public repos work with just `curl` (no `gh` needed)
