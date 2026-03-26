# Session Log

Newest first. Update at the end of every session.

---

## 2026-03-26 — MC — Fix non-marketplace plugin install + session continuity

### What was done
- Diagnosed why `/quick-install michaelcarwile/claude-session-recover` failed — skill required `marketplace.json` but target repo only has `plugin.json`
- Investigated git history (4 commits) — confirmed no version ever had fallback logic
- Reviewed context from another machine's session where the same failure was encountered
- **Fix**: Swapped `marketplace.json` for `plugin.json` as primary metadata source in step 2 — same three fields (name, description, version), simpler data shape (flat vs nested array)
- **Added Tier 3**: Standalone skill repo detection — checks for `skills/*/SKILL.md` when no `.claude-plugin/` exists, installs via file copy to `~/.claude/skills/`
- Updated README: git clone as primary install method, curl as backup. Updated How It Works and Requirements
- Copied updated SKILL.md to installed location (`~/.claude/skills/quick-install/`)
- Built SESSIONLOG.md, CHANGELOG.md, and AGENTS.md following the bokteachings/gdrive-kb-mcp pattern

### Current state
- All changes committed and pushed: `d0d8457` (fix), `e0f1c36` (docs)
- Installed copy at `~/.claude/skills/quick-install/SKILL.md` is updated and matches source
- Not yet tested end-to-end with `/quick-install`

### What's next
- Test `/quick-install michaelcarwile/claude-session-recover` (plugin.json path)
- Test `/quick-install michaelcarwile/claude-skill-quick-install` (standalone skill path)
- Commit and push
- Apply same fix on other machine

### Open questions / blockers
- Monorepo support (repos with multiple plugins in `marketplace.json`) not handled — single `plugin.json` assumed. Edge case, not blocking.

---

## 2026-03-23 — MC — Convert from plugin to standalone skill

### What was done
- Removed `.claude-plugin/` directory (plugin.json, marketplace.json)
- Removed `commands/quick-install.md`
- Kept only `skills/quick-install/SKILL.md` — skill is self-contained
- Rationale: quick-install exists to avoid the marketplace bootstrap dance; requiring a marketplace to install it was self-defeating

### Current state
- Standalone skill at `skills/quick-install/SKILL.md`
- Installable via curl or git clone to `~/.claude/skills/quick-install/`

### What's next
- Install and test on multiple machines
- Handle repos without `marketplace.json` (known gap)

### Open questions / blockers
- None

---

## 2026-03-14 — MC — Initial build

### What was done
- Built quick-install as a Claude Code plugin with local marketplace approach
- Implemented `curl` → `gh api` fallback chain for fetching marketplace manifest
- Added README with install instructions
- Fixed `gh` dependency — switched to `curl` as primary fetch method for public repos

### Current state
- Working plugin for repos that have `.claude-plugin/marketplace.json`
- Published to GitHub as `michaelcarwile/claude-skill-quick-install`

### What's next
- Convert to standalone skill (marketplace bootstrap is self-defeating)
- Handle repos without marketplace manifest

### Open questions / blockers
- Repos without `marketplace.json` can't be installed (known limitation)
