---
name: www-commit
description: Trigger whenever the user asks to commit AND wants the commit message enriched with AI collaboration history. Records key prompts verbatim, structured assessment (goal/outcome/friction), and summarizes the AI collaboration journey since the last commit as a collapsible <details> block on GitHub/GitLab PRs. The key signal is the combination of (1) making a commit with (2) capturing how AI contributed. Trigger on phrases like commit with AI recap, attach collaboration history to commit, record AI work in commit, www-commit. DO NOT trigger for plain commits without AI documentation, standalone time-period recaps (use www-insights instead), PR reviews, or general git operations.
allowedTools:
  - Bash
  - Read
  - Glob
  - Grep
  - Write
  - Edit
---

You are creating a git commit that includes an **AI collaboration recap** — a record of how the developer collaborated with AI to produce this commit's changes.

## Step 1: Run the Engine

```bash
ENGINE=/Users/taehyoungkim/Documents/DEV/ww-w-ai/www-cowork/src/cli.ts
bun run "$ENGINE" www-commit --path "$PWD"
```

This scans sessions since the last git commit **in the current project** and outputs JSON
with metrics and user prompts. Do NOT `cd` into the plugin dir — the engine must target
the user's repo via `--path`.

- If output contains `{"error": "no_previous_commit"}`, tell the user to use `/commit` instead.
- If `sessions` is 0, tell the user no sessions were found since the last commit.

## Step 2: Select Key Prompts (2-3)

From the `userPrompts` array, pick **2-3 that most influenced the outcome**:

1. **Direction changes**: "actually, let's use X instead"
2. **Key decisions**: "let's use TypeScript", "split this into..."
3. **Root causes**: "the real problem is..."
4. **Crucial context**: "our API expects..."

Copy text **exactly as written** — typos, mixed languages, informal tone preserved.

Skip routine prompts: "ok", "yes", "continue", "fix that error", "looks good"

## Step 3: Structured Assessment

| Field | Options |
|-------|---------|
| **Goal** | Free text |
| **Outcome** | `fully_achieved` / `mostly_achieved` / `partially_achieved` / `not_achieved` |
| **AI Helpfulness** | `essential` / `very_helpful` / `moderately_helpful` / `slightly_helpful` / `unhelpful` |
| **Friction** | `misunderstood_request` / `wrong_approach` / `buggy_code` / `user_rejected_action` / `excessive_changes` / `none` |

## Step 4: Format the Recap Block

```
<details>
<summary>AI Recap — N sessions, X.Xh, M messages, +A/-R lines</summary>

## Summary
[2-3 sentences]

## Key Prompts

**Prompt 1**: [why pivotal]
> [exact user prompt, verbatim]

-> [what AI did, 1-2 sentences]

**Prompt 2**: [why pivotal]
> [exact user prompt, verbatim]

-> [what AI did, 1-2 sentences]

## Other Prompts
[1 paragraph grouping by theme with counts]

## AI Contribution
- **Tools**: [top tools with counts and pattern description]
- **Approach**: [how AI tackled this]
- **Friction**: [what went wrong, or "None"]

## Assessment
- **Goal**: [what user wanted]
- **Outcome**: [level]
- **AI Helpfulness**: [level]
- **Friction**: [types or "None"]

## Learnings
[1-2 sentences]

## Stats
- Sessions: N | Messages: M | Duration: X.Xh
- Tools: [top 5]
- Lines: +A / -R

</details>
```

Keep the block under ~1500 words. Key prompts are the most valuable part.

## Step 4.5: Write the directive-log file (forward)

Run the engine to get full-depth directives since the last commit:

```bash
ENGINE=/Users/taehyoungkim/Documents/DEV/ww-w-ai/www-cowork/src/cli.ts
bun run "$ENGINE" commit-log --path "$PWD"
```

This outputs `{ window, turns }`. Each turn has `ts`, `sessionId`, `line`, `userText`,
`isReactive`, optional `precedingAssistant` / `options` / `decision`.

1. If `turns` is empty, skip the file (still create the commit with the message block).
2. **`slug`** — a SHORT ENGLISH descriptor coined from the commit subject's meaning,
   lowercase, words joined by `-`, ≤ 50 chars. Do NOT mechanically slugify a Korean subject.
3. **timestamp** — "now" in KST as `YYYYMMDD-HHMMSS`:
   ```bash
   TS=$(TZ=Asia/Seoul date +%Y%m%d-%H%M%S)
   ```
4. **filename** = `docs/commit-log/$TS-$slug.md`. If that file already exists, append `-2`.
5. Write the file using the template in `references/commit-log-format.md`:
   - **리캡 섹션** (상단): Step 1의 engine 출력에서 세션 수, 시간, 메시지 수, 도구 top 3,
     라인 변경량을 표로 채우고, 2-3문장 요약 + 마찰 항목 작성.
   - **지시 이력 섹션**: verbatim user text, 🤖 assistant context on reactive turns,
     `[sessionId Ln]` source markers, in `turns` order.
   - Header `일시(KST)` = `TZ=Asia/Seoul date "+%Y-%m-%d %H:%M:%S"`.
6. Update `docs/commit-log/README.md` — append one row; create the file with header if absent.
7. Stage both: `git add "docs/commit-log/$TS-$slug.md" docs/commit-log/README.md`

The file rides in the SAME commit created by Step 5.

## Step 5: Create the Commit

1. Run `git status` and `git diff HEAD`
2. Stage relevant files (prefer specific files over `git add -A`)
3. Commit with HEREDOC:

```bash
git commit -m "$(cat <<'EOF'
<concise commit title>

<recap block from Step 4>
EOF
)"
```

Do not stage secrets (.env, credentials). Do not create empty commits.

## Backfill mode (document past commits)

Triggered when the user asks to backfill / document existing commits.

```bash
ENGINE=/Users/taehyoungkim/Documents/DEV/ww-w-ai/www-cowork/src/cli.ts
```

1. List commits: `git log --format='%H %cI %s' --no-merges`.
2. Read existing `docs/commit-log/` filenames; parse `YYYYMMDD-HHMMSS` prefixes → documented timestamps.
3. Undocumented = commits with no matching doc. If > 8, ask the user which to document.
4. For each target commit C (committer time `C_cI`) with previous commit P (`P_cI`):
   ```bash
   bun run "$ENGINE" commit-log --path "$PWD" --from "$P_cI" --to "$C_cI"
   ```
   For the first commit (no P), omit `--from`.
5. Filename from C's committer time in KST:
   ```bash
   TS=$(TZ=Asia/Seoul date -j -f '%Y-%m-%dT%H:%M:%S%z' "${C_cI}" +%Y%m%d-%H%M%S 2>/dev/null)
   ```
6. Write `docs/commit-log/$TS-<slug>.md` (slug = short English descriptor). On collision append `-2`.
   The commit hash MAY be included in the body (commit already exists — no paradox).
7. Update `docs/commit-log/README.md` (time order, append-only).
8. Commit: `git add docs/commit-log/ && git commit -m "docs(commit-log): backfill <range>"`
