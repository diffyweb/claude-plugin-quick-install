---
name: quick-install
description: >
  Install a Claude Code plugin directly from a GitHub repo in one command.
  Use when the user says: "quick install", "install plugin", "install from github",
  or provides a GitHub owner/repo or URL to install.
---

# /quick-install — One-Command Plugin Installation

Install a Claude Code plugin from a GitHub repository without manually managing marketplaces. Uses a single local marketplace (`quick-install`) to avoid clutter.

## Input

`$ARGUMENTS` is either:
- `owner/repo` format (e.g., `michaelcarwile/claude-skill-quick-install`)
- A full GitHub URL (e.g., `https://github.com/michaelcarwile/claude-skill-quick-install` or `https://github.com/michaelcarwile/claude-skill-quick-install.git`)

## Steps

### 1. Parse the input

Extract `owner/repo` from the arguments. If a full URL is given, strip the `https://github.com/` prefix and any `.git` suffix.

### 2. Fetch plugin metadata from the repo

Try `curl` first (works for public repos, no dependencies):

```bash
curl -sf "https://raw.githubusercontent.com/$OWNER_REPO/main/.claude-plugin/marketplace.json" | python3 -c "
import sys, json
data = json.load(sys.stdin)
for p in data['plugins']:
    print(json.dumps({'name': p['name'], 'description': p.get('description', ''), 'version': p.get('version', '1.0.0')}))
"
```

If curl returns a 404, try `master` or `HEAD` as the branch instead of `main`.

If curl fails on all branches, the repo may be private. Fall back to `gh` (requires GitHub CLI with auth):

```bash
gh api "repos/$OWNER_REPO/contents/.claude-plugin/marketplace.json" --jq '.content' | base64 -d | python3 -c "
import sys, json
data = json.load(sys.stdin)
for p in data['plugins']:
    print(json.dumps({'name': p['name'], 'description': p.get('description', ''), 'version': p.get('version', '1.0.0')}))
"
```

If `gh` is not installed, tell the user:
> "This repo appears to be private. Install the GitHub CLI (`gh`) and authenticate with `gh auth login` to install private plugins."

If all methods fail, tell the user:
> "This repo doesn't have a `.claude-plugin/marketplace.json` manifest. It needs one to be installable as a Claude Code plugin. See https://code.claude.com/docs/en/plugin-marketplaces for the required format."

Stop here if it fails.

### 3. Ensure the `quick-install` local marketplace exists

Check if `~/.claude/quick-install-marketplace/.claude-plugin/marketplace.json` exists.

If it does NOT exist, create it:

```bash
mkdir -p ~/.claude/quick-install-marketplace/.claude-plugin
```

Write this to `~/.claude/quick-install-marketplace/.claude-plugin/marketplace.json`:

```json
{
  "name": "quick-install",
  "owner": {"name": "quick-install"},
  "metadata": {"description": "Plugins installed via /quick-install", "version": "1.0.0"},
  "plugins": []
}
```

Then register it:

```bash
claude plugin marketplace add ~/.claude/quick-install-marketplace
```

### 4. Add the plugin entry to the local marketplace

Read `~/.claude/quick-install-marketplace/.claude-plugin/marketplace.json`, check if a plugin with this name already exists in the `plugins` array. If it does, update its source URL. If not, append a new entry:

```json
{
  "name": "$PLUGIN_NAME",
  "description": "$PLUGIN_DESCRIPTION",
  "version": "$PLUGIN_VERSION",
  "source": {
    "source": "url",
    "url": "https://github.com/$OWNER_REPO.git"
  }
}
```

Write the updated marketplace.json back to disk.

### 5. Refresh and install

```bash
claude plugin marketplace update quick-install
claude plugin install "$PLUGIN_NAME@quick-install"
```

If there are multiple plugins in the repo, ask the user which ones they want, or install all if they say so.

### 6. Reload

Tell the user to run `/reload-plugins` to activate the new plugin in the current session.

## Output

Report what happened:
- Plugin name(s) installed
- Added to the `quick-install` marketplace
- Remind to run `/reload-plugins`

## Examples

```
/quick-install jarrodwatts/claude-hud
/quick-install michaelcarwile/claude-skill-quick-install
/quick-install https://github.com/EveryInc/compound-engineering-plugin
```
