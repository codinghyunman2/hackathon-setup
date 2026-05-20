# Clarification Question Pool

Used in Step 2 of the workflow. Pick up to 3 of the most impactful questions per round, max 2 rounds. Always rewrite the question to reference the user's specific situation — never copy a generic template verbatim.

Each question below carries 2-3 example options so the AskUserQuestion call has concrete answers the user can pick. Always allow free-text input via the implicit "Other" option.

## Contents

- Group A — Problem clarity (A1 frequency, A2 time cost, A3 workaround, A4 as-is workflow ★ Round 1 필수)
- Group B — Output expectation (B1 form, B2 destination, B3 cadence)
- Group C — Scope & users (C1 user scope, C2 headcount, C3 deployment)
- Group D — Data & integrations (D1 source, D2 auth, D3 tools)
- Group F — Feasibility (F1 access permission ★ Round 1 필수, F2 build method)
- Group E — Constraints (E1 hard no, E2 time budget, E3 help available)
- Question selection heuristic (Round 1 mandatory items + Round 1 remaining slot priority)

---

## Group A — Problem clarity (use when "문제" section is thin)

### A1. Real frequency
**Question**: "이 문제가 실제로 얼마나 자주 발생하나요?"
- 매일 — 매일 1회 이상 반복돼요
- 매주 — 주 1~2회 정도
- 매월 또는 비정기 — 월말이나 이벤트 있을 때만

### A2. Time cost
**Question**: "한 번 처리할 때 보통 얼마나 걸려요?"
- 30분 미만 — 짧지만 자주라 피곤해요
- 30분~2시간 — 한나절이 사라져요
- 반나절 이상 — 다른 일을 못 해요

### A3. Current workaround
**Question**: "지금은 어떻게 해결하고 계세요?"
- 엑셀/구글시트 수작업
- 다른 사람한테 부탁
- 그냥 참고 안 함

### A4. As-is workflow steps (★ Round 1 필수 — 자동화 진입점 식별의 핵심)
**Question**: "지금 손으로 하는 단계를 3~5개로 나눠 알려주세요. 짧게 써도 괜찮아요." (free-text 권장; AskUserQuestion 옵션은 가이드용)
- 예시 1) "① Meta Ads에서 CSV 다운 → ② 구글시트 A에 붙여넣기 → ③ 함수로 가공 → ④ 슬랙에 복붙"
- 예시 2) "① 고객 문의 메일 확인 → ② 카테고리 분류 → ③ 담당자 지정 → ④ 노션에 기록"
- "단계가 잘 안 떠올라요" — Claude가 짐작해서 초안 제시 후 사용자가 수정
- **Why this matters**: 자동화는 "어디 한 단계를 기계에 맡길 것인가"의 결정. 단계가 안 보이면 /plan이 추측에 의존해서 어색해짐.

---

## Group B — Output expectation (use when "원하는 것" is vague)

### B1. Output form
**Question**: "결과물이 어떤 모양이면 좋겠어요?"
- 파일 (CSV / 엑셀 / PDF / 슬라이드)
- 메시지 (슬랙 / 이메일)
- 화면 (대시보드 / 웹페이지)

### B2. Delivery destination
**Question**: "결과물이 어디로 가야 해요?"
- 내 컴퓨터의 특정 폴더
- 슬랙 채널 또는 DM
- 구글 드라이브 / 노션 / 이메일

### B3. Output cadence
**Question**: "결과물이 언제 만들어졌으면 좋겠어요?"
- 자동으로 정해진 시간에 (예: 매주 월요일 9시)
- 내가 필요할 때 실행
- 데이터가 들어올 때마다 즉시

---

## Group C — Scope & users (use when "사용 범위" is unclear)

### C1. User scope
**Question**: "결국 누가 이걸 쓰게 되나요?"
- 혼자만 — 내 작업만 줄이고 싶어요
- 팀 내부 — 동료들도 같이 써요
- 외부 공개 — 고객/파트너에게도 노출

### C2. Headcount
**Question**: "사용 인원이 대략 몇 명이에요?"
- 1~3명
- 4~10명
- 10명 이상

### C3. Deployment
**Question**: "어디서 실행될까요?"
- 내 노트북 (로컬)
- 사내 서버 또는 클라우드
- 외부 서비스 배포 (Vercel 등)

---

## Group D — Data & integrations (use when "데이터" is blank)

### D1. Data source
**Question**: "데이터를 어디서 가져와요?"
- 파일 (CSV / 엑셀)
- 구글 시트 / 노션 / 에어테이블
- 외부 API (광고 플랫폼, CRM 등)

### D2. Authentication
**Question**: "데이터에 접근하려면 로그인/키가 필요한가요?"
- 네, API 키나 토큰 있어요
- 네, 하지만 어떻게 받는지 몰라요
- 아니요, 공개 데이터예요

### D3. Tool integrations
**Question**: "어떤 툴이 연결돼야 해요?"
- Slack
- Google Workspace (Sheets / Drive / Calendar)
- 광고/분석 플랫폼 (Meta, Google Ads, Airbridge, Amplitude 등)

---

## Group F — Feasibility (★ Round 1 필수 — 6시간 안에 끝낼 수 있는지의 핵심 변수)

### F1. System access permission
**Question**: "이 자동화에 필요한 시스템에 접근 권한이 있어요? (예: Slack 워크스페이스 권한, 구글 시트, 사내 DB, 광고 플랫폼 계정)"
- 네, 모두 있어요 / 받을 수 있어요
- 일부만 있어요 — 추가 권한 신청이 필요해요
- 모르겠어요 / 사내 IT 확인 필요해요
- **Why this matters**: 권한 없으면 6시간 안에 완성 불가. PRD에 자동으로 `[Risk: 접근권한 사전 확인 필요]` 마킹. /plan이 첫 스텝을 권한 확인으로 잡도록 유도.

### F2. Build method preference (Step 3 후반에서 다시 묻지만, 사전에 신호가 있으면 캡처)
**Question**: "혹시 어떤 방식으로 만들고 싶다는 생각이 있어요?"
- Claude Code로 코드 짜고 싶어요
- n8n 같은 노드 연결 툴로 만들고 싶어요
- 잘 모르겠어요 — 추천해주세요
- **Why this matters**: 후속 산출물(`docs/n8n-workflow.md` 생성 여부)을 결정. 사용자가 미정이면 Claude가 PRD 내용 기반 1줄 추천.

---

## Group E — Constraints (use when checking blockers)

### E1. Hard no
**Question**: "절대 안 됐으면 하는 게 있어요?"
- 데이터를 외부 서버에 올리면 안 돼요
- 자동 발송/실행이 부담스러워요 — 수동 확인 필요
- 비용이 발생하면 안 돼요

### E2. Time budget
**Question**: "해커톤에서 며칠 안에 끝내야 해요?"
- 1일 (오늘 안에)
- 2~3일
- 1주일 이상

### E3. Help available
**Question**: "막히면 누가 도와줄 수 있어요?"
- 같은 팀 개발자
- 해커톤 멘토
- 혼자 해결해야 해요

---

## Question selection heuristic

**Round 1 (필수 — 다음 2개는 항상 포함, 답이 이미 Step 1에서 분명하면 skip)**:
1. **A4** — As-is 단계 캡처 (자동화 진입점 식별)
2. **F1** — 시스템 접근 권한 (6시간 안 완성 가능성)

**Round 1 나머지 슬롯 (남은 1개) — 다음 우선순위로 선택**:
3. **B1/B2** — output form and destination (highest impact on /plan accuracy)
4. **A1** — frequency (determines whether automation is worth it)
5. **C1/C3** — user scope and deployment (affects security/architecture)
6. **D1/D3** — data source and integrations (determines tool needs)
7. **E1** — hard constraints (prevents wasted /plan output)
8. **F2** — build method (n8n vs Claude Code) — Step 3에서 다시 확정하지만 사전 신호 있으면 캡처

If the user picks "Other" or writes a vague free-text answer, do not re-ask the same question — move to the next most important gap or accept the ambiguity and add `[가정: ...]` to the PRD.

**Special handling for A4 (As-is steps)**: 만약 사용자가 "단계가 안 떠올라요"라고 답하면, 사용자의 Step 1 입력 + 도메인 상식을 토대로 3~5단계 초안을 직접 작성해서 보여주고 "이 흐름이 맞나요? 틀린 부분만 알려주세요" 형태로 확인. 빈 답으로 진행하지 말 것.
