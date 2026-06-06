# pdca-wf — document output templates (fill, never restructure)

Fixed skeletons for every md artifact pdca-wf writes. Fill the `{slots}`; do not invent/remove sections. (Workflow JSON returns are templated separately in `schemas.md`.)

## Plan — `02-planned/<dt>-<feature>-plan.md` (Phase 2)

```markdown
# {feature} — Plan

> 상태: ACTIVE-PLAN

## 목표 (한 줄)
{what & why}

## 범위
- IN: {…}
- OUT: {…} (명시적 컷)

## Done 기준
- {verifiable criterion 1}
- {…}

## 리스크
| 리스크 | 대응 |
|---|---|
| {…} | {…} |
```

## Design — `02-planned/<dt>-<feature>-design.md` (Phase 3, single input to Do)

```markdown
# {feature} — Design

> 상태: ACTIVE-PLAN

## 접근 (왜 이 방식 — 기각한 대안 포함)
{approach; rejected: {alt} because {…}}

## 변경 지도
| 파일 | 변경 | 이유 |
|---|---|---|
| {path} | {…} | {…} |

## WorkList (machine-readable — Do의 입력)
~~~json
{ "items": [ { "id": "{W1}", "file": "{path}", "change": "{…}", "dependsOn": [] } ] }
~~~

## 검증 방법
- verifyCmd: `{cmd}` (없으면 null + 이유)
- 렌즈: correctness / regression / design-fit (+ {risk-specific lens})

## 미결 (있으면 — 구현 전 잠금 필요 항목만)
- {…}
```

## Check snapshot — `05-reports/<dt2>-<feature>-check.md` (Phase 5)

```markdown
# {feature} — Check {dt2}

| Iter | 실행 체크 (exit code) | 렌즈 | matchRate | 수정한 갭 |
|---|---|---|---|---|
| 1 | {verifyCmd: pass/fail} | {…} | {n}% | {…} |

최종: matchRate {n}% / iterations {n}/5
잔여 갭 (max 5 소진 시만): {gap + severity, 없으면 "없음"}
```

## Report — `05-reports/<dt3>-<feature>-report.md` (Phase 6)

```markdown
# {feature} — Report {dt3}

## 결과 (제품 언어 한 줄)
{what the user can now do}

## QA 표
| 동작 | 무엇으로 증명 | 상태 |
|---|---|---|
| {…} | {runner/probe/live} | ✅ / ⏸ deferred — {이유} |

## phaseHistory (Check 반환값 passthrough — LLM 재구성 금지)
iterations: {n} / testsRun: {n} / matchRate: {n}%

## 잔여 & carry
| 항목 | 사유 | 다음 |
|---|---|---|
| {… or "없음"} | | |

## 예상 질문 선제 답변
- **Q. {…}?** — {답}
```

## 01-built section (Phase 6 merge — section-scoped, never whole-file)

```markdown
## {feature}  <!-- 최종 갱신: {YYYY-MM-DD HH:MM} -->
{as-built: 현재 진실만, 과거형/취소선 금지. superseded 내용은 삭제(git이 보존).}
- 동작: {…}
- 위치: {files}
- 제약/불변식: {…}
```
