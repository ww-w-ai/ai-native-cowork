# Sprint Consolidated Report — {initiative} ({date})

> 상태: FROZEN (작업 스냅샷). {N} sprints / {M} commits ({first}…{last}) / 배포: {summary}.

<!-- RULE: 독자(사용자) 기준으로 작성. 읽은 직후 "그래서 X는?"이 나오면 §4 실패.
     코드 식별자보다 화면/제품 언어 우선. -->

## 1. 스프린트별 결과

| Sprint | What shipped (제품 언어 한 줄) | QA | Commit |
|---|---|---|---|
| {SP-1} | {…} | PASS / PASS+fixed({n}) / deferred | `{hash}` |

적대 검증에서 잡혀 커밋 전 수정된 결함: {n}건 — {한 줄 목록, 없으면 "없음"}

## 2. QA 커버리지 표 (sprints[].qaTable 통합)

<!-- RULE: 출하된 기능/동작마다 1행. 체크 안 된 행에 사유 없으면 QA 게이트 FAIL이었어야 함. -->

| Feature/behavior | 무엇으로 증명 | 상태 | 비고 |
|---|---|---|---|
| {…} | test runner `{name}` / manual probe / live check | ✅ checked | |
| {…} | deferred-to-deploy | ⏸ deferred | {이유: 실키/실환경 필요 등} |

요약: checked {n} / deferred {n} / total {n}

## 3. Pending gates — 막혀 있는 것과 푸는 법

| Gate | 내용 | 무엇이 풀어주나 |
|---|---|---|
| 배포 | {마이그/워커/큐/재배포 목록} | 사용자 배포 승인 세션 |
| 외부 의존 | {API 키 발급 등} | {…} |
| 재색인 등 후속 | {…} | {…} |

## 4. 예상 질문 선제 답변

<!-- RULE: "이 리포트를 읽은 사용자가 바로 물을 질문"을 3~5개 스스로 뽑아 답한다.
     후보 채굴: QA 깊이("라이브로 검증됐나?"), 비용, 미배포 이유, 다음 순서, 리스크. -->

- **Q. {…}?** — {답}
- **Q. {…}?** — {답}

## 5. Carry items

| Item | 명시적 사유 | 다음 |
|---|---|---|
| {…} | {…} | {…} |

## 6. 다음 액션 제안

1. {…} (권장 — {이유 한 줄})
2. {…}
