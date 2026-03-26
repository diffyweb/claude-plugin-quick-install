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
curl -sf "https://raw.githubusercontent.com/$OWNER_REPO/main/.claude-plugin/plugin.json" | python3 -c "
import sys, json
data = json.load(sys.stdin)
print(json.dumps({'name': data['name'], 'description': data.get('description', ''), 'version': data.get('version', '1.0.0')}))
"
```

If curl returns a 404, try `master` or `HEAD` as the branch instead of `main`.

If curl fails on all branches, the repo may be private. Fall back to `gh` (requires GitHub CLI with auth):

```bash
gh api "repos/$OWNER_REPO/contents/.claude-plugin/plugin.json" --jq '.content' | base64 -d | python3 -c "
import sys, json
data = json.load(sys.stdin)
print(json.dumps({'name': data['name'], 'description': data.get('description', ''), 'version': data.get('version', '1.0.0')}))
"
```

If `gh` is not installed, tell the user:
> "This repo appears to be private. Install the GitHub CLI (`gh`) and authenticate with `gh auth login` to install private plugins."

If all methods fail, this repo may be a standalone skill instead of a plugin. Proceed to **Step 2b**.

### 2b. Check for standalone skill repos (no `.claude-plugin/` directory)

If step 2 found no `plugin.json`, check whether the repo contains skills directly.

Fetch the repo tree:

```bash
curl -sf "https://api.github.com/repos/$OWNER_REPO/git/trees/main?recursive=1" | python3 -c "
import sys, json
tree = json.load(sys.stdin).get('tree', [])
skills = [e['path'] for e in tree if e['path'].endswith('/SKILL.md') and e['path'].startswith('skills/')]
for s in skills:
    print(s)
"
```

If no skills found, try `master` branch. If still nothing, fall back to `gh api` for private repos.

If skill paths are found (e.g., `skills/my-skill/SKILL.md`):

1. For each skill, extract the skill directory name (e.g., `my-skill` from `skills/my-skill/SKILL.md`)
2. Download the skill directory to `~/.claude/skills/<skill-name>/`:

```bash
# For each skill path found:
mkdir -p ~/.claude/skills/$SKILL_NAME
curl -sf "https://raw.githubusercontent.com/$OWNER_REPO/main/$SKILL_PATH" \
  -o ~/.claude/skills/$SKILL_NAME/SKILL.md
```

3. Tell the user:
> "Installed `<skill-name>` as a standalone skill at `~/.claude/skills/<skill-name>/`. Restart Claude Code or start a new session to activate it."

**Stop here** — standalone skills don't use the marketplace flow, so skip steps 3-6.

If no `plugin.json` AND no `skills/*/SKILL.md` found, tell the user:
> "This repo doesn't have `.claude-plugin/plugin.json` or a `skills/` directory. It can't be installed as a Claude Code plugin or skill."

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
