# ai-native-cowork

Claude Code plugin that records AI collaboration history per commit.

## Project Overview

- **Type**: Claude Code plugin (skills + TypeScript engine)
- **Version**: 1.5.0
- **Runtime**: Bun (TypeScript, no package.json — uses manifest.json)
- **Skills**: `/cowork-commit` (per-commit AI recap), `/cowork-insights` (session reports)

## Architecture

```
manifest.json              # Plugin metadata (name, version, skills, scripts)
.claude-plugin/plugin.json # CC plugin registry entry
skills/
  cowork-commit/SKILL.md   # Commit skill instructions
  cowork-insights/SKILL.md # Insights skill instructions
src/
  cli.ts                   # Entry point — subcommands: cowork-commit, commit-log, cowork-insights
  recap-engine.ts          # Session scanner + metrics aggregator
  session-scanner.ts       # ~/.claude/projects/ JSONL discovery
  commit-log.ts            # Conversation turn extraction from JSONL
  metrics-extractor.ts     # Tool counts, tokens, response times from JSONL
  facet-cache.ts           # Per-session metrics cache
  generate-narrative.ts    # cowork-insights narrative builder
  html-report.ts           # HTML report renderer
docs/commit-log/           # Directive-log files (conversation + recap per commit)
evals/                     # Skill trigger accuracy tests
```

## Key Patterns

- Engine runs via `bun run src/cli.ts <subcommand> --path <target-repo>`
- Engine targets the user's repo, not this plugin dir — always pass `--path "$PWD"`
- Session discovery: scans `~/.claude/projects/<hash>/` for JSONL transcripts
- Project hash = `path.replace(/[^a-zA-Z0-9]/g, '-')` — directory renames break hash linkage
- Facet cache: `~/.claude/recap-data/facet-cache/` — per-session metrics JSON, keyed by sessionId

## Running Tests

```bash
bun test src/commit-log.test.ts
```

## Commit Language

All commit messages, directive-log recaps, and documentation in **English**.

```
/cowork-commit --language en
```
