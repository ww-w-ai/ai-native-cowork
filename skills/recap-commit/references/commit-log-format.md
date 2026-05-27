# commit-log file format

Output: `docs/commit-log/<YYYYMMDD-HHMMSS>-<slug>.md` (timestamp = the documented commit's
committer time, KST; forward mode uses "now" at write time). **No commit hash in the
filename or body for forward mode** (a commit can't contain its own hash). Backfill MAY put
the hash in the body since those commits already exist.

`slug` = a SHORT ENGLISH descriptor coined from the commit subject's meaning, lowercase,
words joined by `-`, ≤ 50 chars. Commit subjects here are usually Korean; a mechanical
`[^a-zA-Z0-9]→-` slug would strip Korean to empty, so coin a meaningful English slug instead
(e.g. "런타임 노이즈 gitignore 추가" → `gitignore-noise`).

## Template

```markdown
# <commit subject>

- **일시(KST)**: YYYY-MM-DD HH:MM:SS
- **세션**: `<sessionId>`[, `<sessionId>` …]

> 시간순 verbatim 전사. `>` = 유저 원문. 🤖 = 직전 어시스턴트(축약, 유저가 응답한 경우만).

---

**HH:MM [<sessionId> L<line>]**
> <userText, verbatim>

**HH:MM [<sessionId> L<line>]** — 어시스턴트 제안에 응답
- 🤖 *"<precedingAssistant, truncated>"*
- 🤖 *[선택지] <options joined by " / ">*   ← only when options present
> **[선택] → "<decision>"**                ← only when decision present
> <userText if it adds beyond the decision>
```

## Rules
- Emit turns in `turns[]` order (already sorted by `ts`).
- For non-reactive turns: just the `>` user line with its `[sessionId Ln]` marker.
- For reactive turns (`isReactive: true`): show the 🤖 preceding-assistant line; if `options`
  present, add the `[선택지]` line; if `decision` present, add the `[선택]` line.
- Keep user text verbatim — do not paraphrase. Assistant text may stay truncated as provided.
- Group consecutive turns under a section heading only if it aids reading; otherwise list
  them flat in time order.

## README index (`docs/commit-log/README.md`)
Maintain a table; append/update one row per doc. Never rewrite existing rows.

```markdown
| 일시(KST) | 요지 | 문서 |
|-----------|------|------|
| YYYY-MM-DD HH:MM:SS | <commit subject> | [<slug>](./<filename>) |
```
