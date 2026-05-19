# Clarification Question Pool

Used in Step 2 of the workflow. Pick up to 3 of the most impactful questions per round, max 2 rounds. Always rewrite the question to reference the user's specific situation — never copy a generic template verbatim.

Each question below carries 2-3 example options so the AskUserQuestion call has concrete answers the user can pick. Always allow free-text input via the implicit "Other" option.

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

For each round, prioritize this order:
1. **B1/B2** — output form and destination (highest impact on /plan accuracy)
2. **A1** — frequency (determines whether automation is worth it)
3. **C1/C3** — user scope and deployment (affects security/architecture)
4. **D1/D3** — data source and integrations (determines tool needs)
5. **E1** — hard constraints (prevents wasted /plan output)

If the user picks "Other" or writes a vague free-text answer, do not re-ask the same question — move to the next most important gap or accept the ambiguity and add `[가정: ...]` to the PRD.
