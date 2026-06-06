# Sprint Retrospective — {initiative} ({date})

> 상태: FROZEN (회고 스냅샷). cowork-sprint PHASE 2 산출 — PROPOSALS ONLY, 적용은 apply-gate 선택분만.
> {N} sprints, {M} commits ({first}…{last}). 배포: {deployed|로컬 커밋만}.

<!-- RULE: every section ends in an evolution proposal OR an explicit "no change needed — {why}".
     Do NOT re-summarize the work here (that is the consolidated report's job). -->

## 1. Agent evolution (A)

| Agent (owned scaffolds only) | Dispatches | QA pass | Fix rounds | Unblock evolutions | Verdict |
|---|---|---|---|---|---|
| {name or "(none scaffolded)"} | {n} | {n}/{n} | {n} | {n}/2 | keep / evolve / split / retire |

- Scaffolded agents = 0 인 경우 → 아래 판단을 반드시 답할 것:
  - 재사용(general-purpose/플러그인)으로 충분했나? {yes — why / no — which role should have been scaffolded}
  - 다음 런 스캐폴드 후보: {role name + one-line charter, or "없음"}
- DEFINITION-defect diffs (recurring만): {agent}.md — {exact change}, or "없음"

## 2. Self-assessment → evolution backlog (B)

### 잘된 것
| # | What went well | Evidence (signal/number) |
|---|---|---|
| 1 | {…} | {…} |

### 못한 것 (friction 신호에서 채굴: 사용자 정정/인터럽트, 리포트 직후 질문, 늦게/안 터진 게이트)
| # | What went poorly | Evidence | Tag | Target file (FIXABLE만) | Proposed change |
|---|---|---|---|---|---|
| 1 | {…} | {user quote / event} | [FIXABLE-PROMPT] | {SKILL.md §…/agent.md} | {one-line diff intent} |
| 2 | {…} | {…} | [FIXABLE-SCRIPT] | {script/schema path} | {…} |
| 3 | {…} | {…} | [PROCESS] | — | → Carry/open question |

## 3. Lessons → rule promotion (C)

| # | Non-obvious learning | Promote to rule/template? | Where |
|---|---|---|---|
| 1 | {…} | yes / no — {why} | {SKILL.md §… / templates/… / docs/00-reference/…} |

- Reusable-asset promotion candidates: {prompt/script/agent → destination, or "없음"}

## 4. Carry (E) — 실행 잔여만 (진화 항목 섞지 말 것)

| Carry item | 왜 지금 못 끝냄 (explicit) | 다음 |
|---|---|---|
| {…} | {…} | {…} |

---

## APPLY-GATE — 아래 번호로 선택 (예: "1,3 적용" / "전부" / "보류")

<!-- RULE: 사용자에게 이 표 그대로 제시. 각 항목 = 위 섹션의 FIXABLE/promotion/scaffold 제안 1:1. 승인분만 적용. -->

| # | Proposal | Source | Target file | Effort |
|---|---|---|---|---|
| 1 | {…} | B-1 | {…} | S/M/L |
| 2 | {…} | C-1 | {…} | S |

미응답/인터럽트 시: 이 표를 Carry로 기록하고 종료 (silent drop 금지).
