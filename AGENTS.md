# claude-skill-quick-install

One-command Claude Code plugin/skill installer from GitHub repos.

## What This Is

A standalone Claude Code skill (`/quick-install`) that installs plugins and skills from GitHub without manually managing marketplaces. Uses a local marketplace (`quick-install`) for plugin repos, and direct file copy for standalone skill repos.

## Project Structure

```
SKILL.md                        # The skill itself (all logic lives here)
README.md                       # Install instructions + usage
CHANGELOG.md                    # Commit-level changelog with hashes
SESSIONLOG.md                   # Session handoff log (see Session Continuity below)
AGENTS.md                       # Project instructions (this file)
CLAUDE.md                       # Symlink → AGENTS.md
```

## How It Works

Three tiers of repo detection:

1. **Plugin repos** (`.claude-plugin/plugin.json`): Fetch metadata → create local marketplace entry → `claude plugin install`
2. **Standalone skill repos** (`skills/*/SKILL.md`, no `.claude-plugin/`): Detect skills in repo tree → copy to `~/.claude/skills/`
3. **Neither**: Error with guidance

## Key Design Decisions

- **Standalone skill, not a plugin**: Installing quick-install via a marketplace would be self-defeating. It's a single SKILL.md file installed via `git clone` or `curl`.
- **`plugin.json` over `marketplace.json`**: Every plugin has `plugin.json`; not all have `marketplace.json`. Simpler data shape (flat vs nested array).
- **Local marketplace pattern**: `claude plugin install` requires a marketplace. We maintain one at `~/.claude/quick-install-marketplace/` for all plugins installed via `/quick-install`.
- **`curl` → `gh` fallback**: Public repos use raw GitHub URLs (no dependencies). Private repos fall back to `gh api` (requires GitHub CLI auth).

## Development

This is a skill-as-code project — all logic is in the SKILL.md markdown file that Claude interprets at runtime. There's no build step, no tests, no dependencies.

To test changes: edit `SKILL.md` and run `/quick-install` in a Claude Code session. If installed via `git clone`, the file is already in the right place.

## Session Continuity

If `SESSIONLOG.md` exists, read it at the start of every session. It contains session handoff notes — what was done, current state, and what's next.

**At session end**, update `SESSIONLOG.md`:
- Add a new entry at the top (below the header) with format: `## DATE — INITIALS — Scope`. Include what was done, current state, what's next, and any open questions/blockers
- Keep entries concise — bullets, not prose
- Keep all entries — history is cheap and useful for tracing when things happened
- If the file gets unwieldy, add an `## Archive` section and fold older entries below it
- If the file doesn't exist, carry on without it

Also update `CHANGELOG.md` if commits were made — add entries under `[Unreleased]` or a new date section as appropriate.

If the `/session-search` skill is installed, use it to pull additional context when needed (e.g., searching for decisions or work from sessions older than what `SESSIONLOG.md` covers).
