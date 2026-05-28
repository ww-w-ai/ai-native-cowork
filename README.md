# ai-native-cowork

> **The work-collaboration harness for Claude Code.** Your AI sessions already hold the
> full story of how the work got done — ai-native-cowork turns that history into shareable
> reports and per-commit directive logs your whole team can learn from.

You shipped a week of work with Claude. Friday comes — what did you actually do, what
worked, and *why* did each commit happen? The transcripts know. Nothing else does.

**ai-native-cowork reads your own session history and gives the answer back** — as a polished
report, and as a verbatim record attached to every commit. No new note-taking. No
standup busywork. It runs entirely on data you already have.

| Capability | What you get | Status |
|------------|--------------|:------:|
| **`/cowork-insights`** | Narrative HTML + Markdown report of any time range — key prompts verbatim, per-session assessment, tool/token/cost charts. Paste into Jira, Notion, Slack. Weekly/monthly reviews. | ✅ shipped |
| **`/cowork-commit`** | Two artifacts in one commit: ① `<details>` recap block in the commit message (key prompts + assessment) ② verbatim directive-log file under `docs/commit-log/` with mini-recap stats + full instruction transcript. Backfill mode documents past commits. | ✅ shipped |
| **www-wiki / taise integration** | When the [www-wiki] vault or [taise] harness is installed, cowork-insights output and directive logs file themselves into your knowledge base. Standalone otherwise. | 🛠 planned |

## Why it's different

Most "AI usage" tools count tokens. ai-native-cowork preserves **the prompts, verbatim** — the
actual decisions, pivots, and root-cause insights — because that's the part a teammate can
learn from. It's a clean-room reimplementation of Claude Code's internal `insights`, with
date ranges, per-session assessments, cost ROI, multi-language output, and Markdown built
for sharing. Reports render from a **template engine** (LLM produces structured JSON, the
engine renders HTML+MD), so output is consistent and fast — not LLM-written HTML.

Origin: built in a single Claude Code session, validated across 3 A/B evaluation iterations
against the original `insights` command (95 sessions, 5,943 messages, 4.6B tokens of real
collaboration history).

## Install

**Via ww-w-ai marketplace (recommended):**

Add the ww-w-ai marketplace to your Claude Code settings, then enable the plugin:

```jsonc
// ~/.claude/settings.json
{
  "extraKnownMarketplaces": {
    "ww-w-ai": {
      "source": { "source": "github", "repo": "ww-w-ai/marketplace" }
    }
  },
  "enabledPlugins": {
    "ai-native-cowork@ww-w-ai": true
  }
}
```

Restart Claude Code — the plugin downloads automatically.

**Manual install:**

```bash
git clone https://github.com/ww-w-ai/ai-native-cowork.git
# Add to settings.json:
# "ai-native-cowork@local:/path/to/ai-native-cowork": true
```

**Requirements:** [Claude Code](https://claude.ai/claude-code) v2.1.71+ · [Bun](https://bun.sh) (TypeScript engine).

## Usage

```
/cowork-insights                                    # All time, current project
/cowork-insights --from 1w                          # Last 7 days (exact 168h)
/cowork-insights --from 2026-04-01 --to 2026-04-07  # Date range
/cowork-insights --from 1m --scope all              # Last month, every project
/cowork-insights 최근 1주일, 전체 프로젝트, 한국어로   # Natural language, any language
/cowork-commit                             # Recap + directive log in your next commit
/cowork-commit backfill                    # Document past commits retroactively
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
/cowork-insights --from 1w --scope all
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
ai-native-cowork/
├── .claude-plugin/plugin.json   # Plugin manifest
├── manifest.json
├── skills/
│   ├── cowork-insights/SKILL.md     # /cowork-insights — narrative report pipeline
│   └── cowork-commit/
│       ├── SKILL.md             # /cowork-commit — recap + directive log + backfill
│       └── references/
│           └── commit-log-format.md  # Verbatim transcription template
├── src/                         # Bun + TypeScript engine
│   ├── cli.ts                   # CLI entry (scan, summarize, cowork-commit, commit-log, prepare-facets, render-report)
│   ├── commit-log.ts            # Directive extraction: buildTurns, synthetic filters, reactive pairing
│   ├── commit-log.test.ts       # 16 unit tests (bun:test)
│   ├── session-scanner.ts       # Streaming JSONL parser, path matching, date filtering
│   ├── recap-engine.ts          # Pipeline orchestrator
│   ├── metrics-extractor.ts     # Token / tool / cost / concurrency extraction
│   ├── facet-cache.ts           # Facet + meta cache (stale detection)
│   ├── html-report.ts           # Template engine (JSON → HTML/MD)
│   ├── generate-narrative.ts    # /cowork-insights single-entry pipeline
│   └── git-analyzer.ts          # Git log analysis
├── docs/specs/                  # Design specs
└── evals/                       # Skill trigger + A/B evaluation configs
```

## For teams

When everyone runs `/cowork-insights --from 1w` and shares the Markdown:

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
secretary harness) · **ai-native-cowork** (work collaboration). Your reports are yours. Share
them, analyze them, learn from them.*

[www-wiki]: #
[taise]: #
[anthropics/claude-code#19972]: https://github.com/anthropics/claude-code/issues/19972
