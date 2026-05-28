---
name: cowork-commit
description: Trigger whenever the user asks to commit AND wants the commit message enriched with AI collaboration history. Creates a lightweight commit message (key decision highlights + link) and a full directive-log file with conversation transcript + recap. The key signal is the combination of (1) making a commit with (2) capturing how AI contributed. Trigger on phrases like commit with AI recap, attach collaboration history to commit, record AI work in commit, cowork-commit. DO NOT trigger for plain commits without AI documentation, standalone time-period recaps (use cowork-insights instead), PR reviews, or general git operations.
allowedTools:
  - Bash
  - Read
  - Glob
  - Grep
  - Write
  - Edit
---

You are creating a git commit that includes an **AI collaboration recap** — a record of how the developer collaborated with AI to produce this commit's changes.

## Language

The user may specify a language: `/cowork-commit --language ko` (or `en`, `ja`, etc.).
If omitted, match the language the user has been speaking in this conversation.
This affects the **Recap section** (Summary, Friction, Assessment) and the **commit message body**.
The **Conversation Log** is always verbatim — never translate user/assistant text.

## Step 1: Run the Engine

```bash
ENGINE=/Users/taehyoungkim/Documents/DEV/ww-w-ai/ai-native-cowork/src/cli.ts
bun run "$ENGINE" cowork-commit --path "$PWD"
```

This scans sessions since the last git commit **in the current project** and outputs JSON
with metrics and user prompts. Do NOT `cd` into the plugin dir — the engine must target
the user's repo via `--path`.

- If output contains `{"error": "no_previous_commit"}`, tell the user to use `/commit` instead.
- If `sessions` is 0, tell the user no sessions were found since the last commit.

## Step 2: Format the Commit Message

From the `userPrompts` array, pick **2-3 key conversation lines** that reveal the
decision-making — direction changes, pivots, root causes. Summarize each in one line.

```
<details>
<summary>AI Recap — N sessions, X.Xh, M messages, +A/-R lines</summary>

- "user's key decision quote" → what happened as a result
- "another pivotal quote" → outcome

📄 Full log: docs/commit-log/<filename>.md
</details>
```

Keep the commit message body **under 10 lines**. The full conversation and recap
live in the directive-log file (Step 3), not in the commit message.

## Step 3: Write the directive-log file (forward)

Run the engine to get full-depth directives since the last commit:

```bash
ENGINE=/Users/taehyoungkim/Documents/DEV/ww-w-ai/ai-native-cowork/src/cli.ts
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
   - **Conversation Log** (top): verbatim user text, 🤖 assistant context on reactive turns,
     `[sessionId Ln]` source markers, in `turns` order.
   - **Recap** (bottom): from Step 1 engine output — sessions, duration, messages, top 3 tools,
     lines changed as a table, 2-3 sentence summary, friction, and assessment
     (goal/outcome/helpfulness).
   - Header `Date(KST)` = `TZ=Asia/Seoul date "+%Y-%m-%d %H:%M:%S"`.
6. Update `docs/commit-log/README.md` — append one row; create the file with header if absent.
7. Stage both: `git add "docs/commit-log/$TS-$slug.md" docs/commit-log/README.md`

The file rides in the SAME commit created by Step 4.

## Step 4: Create the Commit

1. Run `git status` and `git diff HEAD`
2. Stage relevant files (prefer specific files over `git add -A`)
3. Commit with HEREDOC:

```bash
git commit -m "$(cat <<'EOF'
<concise commit title>

<recap block from Step 2>
EOF
)"
```

Do not stage secrets (.env, credentials). Do not create empty commits.

## Backfill mode (document past commits)

Triggered when the user asks to backfill / document existing commits.

```bash
ENGINE=/Users/taehyoungkim/Documents/DEV/ww-w-ai/ai-native-cowork/src/cli.ts
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
