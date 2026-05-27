# www-cowork

> **The work-collaboration harness for Claude Code.** Your AI sessions already hold the
> full story of how the work got done — www-cowork turns that history into shareable
> reports and per-commit directive logs your whole team can learn from.

You shipped a week of work with Claude. Friday comes — what did you actually do, what
worked, and *why* did each commit happen? The transcripts know. Nothing else does.

**www-cowork reads your own session history and gives the answer back** — as a polished
report, and as a verbatim record attached to every commit. No new note-taking. No
standup busywork. It runs entirely on data you already have.

| Capability | What you get | Status |
|------------|--------------|:------:|
| **`/recap`** | Narrative HTML + Markdown report of any time range — key prompts verbatim, per-session assessment, tool/token/cost charts. Paste into Jira, Notion, Slack. | ✅ shipped |
| **`/recap-commit`** | A collapsible `<details>` block in your commit message: the key prompts that drove the change, structured assessment, stats. Reviewers see how AI contributed. | ✅ shipped |
| **Directive logs** | Per-commit **verbatim transcription** files under `docs/commit-log/` — the exact instructions that led to each commit, re-typable to reproduce the work. | 🛠 in progress |
| **www-wiki / taise integration** | When the [www-wiki] vault or [taise] harness is installed, recap output and directive logs file themselves into your knowledge base. Standalone otherwise. | 🛠 in progress |

## Why it's different

Most "AI usage" tools count tokens. www-cowork preserves **the prompts, verbatim** — the
actual decisions, pivots, and root-cause insights — because that's the part a teammate can
learn from. It's a clean-room reimplementation of Claude Code's internal `insights`, with
date ranges, per-session assessments, cost ROI, multi-language output, and Markdown built
for sharing. Reports render from a **template engine** (LLM produces structured JSON, the
engine renders HTML+MD), so output is consistent and fast — not LLM-written HTML.

Origin: built in a single Claude Code session, validated across 3 A/B evaluation iterations
against the original `insights` command (95 sessions, 5,943 messages, 4.6B tokens of real
collaboration history).

## Install

```bash
# Clone into your dev workspace
git clone <repo> ~/Documents/DEV/ww-w-ai/www-cowork

# Symlink the skills into Claude Code
ln -s ~/Documents/DEV/ww-w-ai/www-cowork/skills/recap        ~/.claude/skills/recap
ln -s ~/Documents/DEV/ww-w-ai/www-cowork/skills/recap-commit ~/.claude/skills/recap-commit

# Restart Claude Code
```

**Requirements:** [Claude Code](https://claude.ai/claude-code) v2.1.71+ · [Bun](https://bun.sh) (TypeScript engine).

## Usage

```
/recap                                    # All time, current project
/recap --from 1w                          # Last 7 days (exact 168h)
/recap --from 2026-04-01 --to 2026-04-07  # Date range
/recap --from 1m --scope all              # Last month, every project
/recap 최근 1주일, 전체 프로젝트, 한국어로   # Natural language, any language
/recap-commit                             # Attach AI recap to your next commit
```

Three report formats — **full** (deep weekly/monthly review), **standard** (mid-week
check-in), **minimal** (daily standup). Auto-selected by volume: 20+ sessions → full,
1–19 → standard.

### Date range

Relative (`1d`, `7d`/`1w`, `2w`, `1m`) = exact duration from now. Absolute
(`2026-04-01`) = midnight in `--tz` (default `Asia/Seoul`).

### Scope & filtering

| Flag | Values | Description |
|------|--------|-------------|
| `--scope` | `default` / `with-subfolder` / `all` | Scan scope (default = current folder) |
| `--path` | folder (repeatable) | Include specific folders |
| `--exclude-path` | string (repeatable) | Skip private/personal projects |
| `--tz` | IANA zone | Timezone for dates + display |
| `--exclude-subagents` | flag | Exclude sub-agent sessions |

## How it works

```
/recap --from 1w --scope all
        │
        ▼
  [1] Engine scans ~/.claude/projects/ for session JSONL (streaming, on disk)
  [2] Extracts metrics: tools, languages, tokens, lines, response times
  [3] Parallel sub-agents read uncached transcripts → structured "facets"
  [4] LLM produces NarrativeData JSON from facets + metrics
  [5] Template engine renders HTML + Markdown
  [6] Reports saved to ~/.claude/recap-reports/
```

The engine reads transcripts **from disk** — your conversation context is never bloated by
raw JSONL. Facets are cached in `~/.claude/recap-data/` and double as long-term history:
trends survive even after the original session files are deleted.

## Architecture

```
www-cowork/
├── .claude-plugin/plugin.json   # Plugin manifest
├── manifest.json
├── skills/
│   ├── recap/SKILL.md           # /recap — narrative report pipeline
│   └── recap-commit/SKILL.md    # /recap-commit — commit-attached recap
├── src/                         # Bun + TypeScript engine
│   ├── cli.ts                   # CLI entry (scan, summarize, recap-commit, render-report, prepare-facets)
│   ├── session-scanner.ts       # Streaming JSONL parser, path matching, date filtering
│   ├── recap-engine.ts          # Pipeline orchestrator
│   ├── metrics-extractor.ts     # Token / tool / cost / concurrency extraction
│   ├── facet-cache.ts           # Facet + meta cache (stale detection)
│   ├── html-report.ts           # Template engine (JSON → HTML/MD)
│   ├── generate-narrative.ts    # /recap single-entry pipeline
│   └── git-analyzer.ts          # Git log analysis
├── docs/specs/                  # Design specs (e.g. commit directive logs)
└── evals/                       # Skill trigger + A/B evaluation configs
```

## For teams

When everyone runs `/recap --from 1w` and shares the Markdown:

- **Standup replacement** — what you did, what blocked you, what's next
- **Sprint retro data** — friction patterns across the team reveal systemic issues
- **Onboarding** — new members see how the team *actually* uses AI, not theory
- **Management visibility** — which projects consumed AI effort, and where it paid off

## Known limitation

Claude Code encodes project paths by replacing non-alphanumeric characters with `-`, so two
non-ASCII folder names of the same length can collide into one hash and mix sessions. CC core
issue [anthropics/claude-code#19972]. Workaround: use distinct ASCII prefixes
(`work-업무자료`, `personal-개인사진`).

## License

MIT

---

*Part of the **ww-w-ai** ecosystem — [www-wiki] (knowledge vault) · [taise] (librarian +
secretary harness) · **www-cowork** (work collaboration). Your reports are yours. Share
them, analyze them, learn from them.*

[www-wiki]: #
[taise]: #
[anthropics/claude-code#19972]: https://github.com/anthropics/claude-code/issues/19972
