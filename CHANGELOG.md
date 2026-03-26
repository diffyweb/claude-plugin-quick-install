# Changelog

All notable changes to claude-skill-quick-install.

## [Unreleased]

Nothing yet.

## 2026-03-26

- **fix**: Swap `marketplace.json` for `plugin.json` as primary metadata source — handles repos without marketplace manifest (`d0d8457`)
- **feat**: Add standalone skill detection — repos with `skills/*/SKILL.md` but no `.claude-plugin/` install via file copy (`d0d8457`)
- **docs**: Update README with git clone as primary install method, updated How It Works and Requirements (`d0d8457`)
- **docs**: Add session continuity files — SESSIONLOG, CHANGELOG, AGENTS.md + CLAUDE.md symlink (`e0f1c36`)

## 2026-03-23

- **refactor**: Convert from plugin to standalone skill — remove `.claude-plugin/` manifests and command file, keep only `skills/quick-install/SKILL.md` (`07b0e4b`)

## 2026-03-14

- **fix**: Use `curl` instead of `gh` for fetching marketplace manifest — works without GitHub CLI for public repos (`c8a7dc7`)
- **docs**: Add README with install instructions and usage examples (`7d722b0`)
- **feat**: Initial quick-install plugin — local marketplace approach, `curl`/`gh` fallback chain, one-command install (`e593584`)
