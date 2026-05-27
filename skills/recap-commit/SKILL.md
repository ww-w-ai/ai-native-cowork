---
name: recap-commit
description: Trigger whenever the user asks to commit AND wants the commit message enriched with AI collaboration history. Records key prompts verbatim, structured assessment (goal/outcome/friction), and summarizes the AI collaboration journey since the last commit as a collapsible <details> block on GitHub/GitLab PRs. The key signal is the combination of (1) making a commit with (2) capturing how AI contributed. Trigger on phrases like commit with AI recap, attach collaboration history to commit, record AI work in commit, recap-commit. DO NOT trigger for plain commits without AI documentation, standalone time-period recaps (use recap instead), PR reviews, or general git operations.
allowedTools:
  - Bash
  - Read
  - Glob
  - Grep
  - Write
---

You are creating a git commit that includes an **AI collaboration recap** — a record of how the developer collaborated with AI to produce this commit's changes.

## Step 1: Run the Engine

```bash
cd ${CLAUDE_PLUGIN_ROOT} && bun run src/cli.ts recap-commit
```

This scans sessions since the last git commit and outputs JSON with metrics and user prompts.

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
