---
name: hackathon-setup
description: Guides a non-developer through a 5-step Korean conversation that produces docs/PRD.md and CLAUDE.md, ready to hand off to /plan. Use for internal Claude Code mini-hackathon participants who need help turning a work-automation idea into a structured spec. Invoked only by the user typing /hackathon-setup (disable-model-invocation is true, so Claude never auto-invokes it). Trigger phrases the user might use include "사내 해커톤 셋업", "해커톤 시작", "프로젝트 셋업 도와줘", "아이디어 정리해줘", "PRD 만들어줘", "자동화 설계서", or "hackathon setup". If the user describes such a situation in natural language without typing the slash command, suggest they run /hackathon-setup themselves rather than auto-launching the workflow.
disable-model-invocation: true
---

> Surface note: `disable-model-invocation` is a Claude Code-specific frontmatter extension. It is ignored on claude.ai and the Claude API surfaces; slash-only invocation is the intended UX for this skill on Claude Code only.

# Hackathon Setup

> Convert a non-developer's automation idea into a PRD and CLAUDE.md in a guided 5-step Korean conversation, then hand off to `/plan`.

## Audience

Non-developers (marketers, PMs, ops, designers) inside an internal Claude Code mini-hackathon. They have a work problem but do not know how to brief Claude. Reduce the "blank prompt" moment by walking them through a template, asking up to 6 clarifying questions, and producing two English files (`docs/PRD.md`, `CLAUDE.md`) that downstream `/plan` and other agents can consume reliably.

All conversation surfaces are Korean. All saved files are English — explain this in one line ("Claude understands English specs more accurately") when the user reaches Step 3.

## Vocabulary mapping (use throughout the run)

To stay non-developer-friendly, surface friendly Korean labels alongside technical names every time a file is named:

- `docs/PRD.md` → "자동화 설계서 (docs/PRD.md)"
- `CLAUDE.md` → "클로드가 기억할 업무 규칙 파일 (CLAUDE.md)"
- `/plan` → "다음 단계: /plan (실행 계획 자동 생성 커맨드)"

## Workflow

Execute these steps in order. Every user question must be issued via the AskUserQuestion tool — never as plain text with A/B/C options.

**Before starting**: read `references/failure_modes.md` once. It catalogs non-happy-path scenarios (one-sentence answers, mid-flow language switches, unsafe requests, recovery commands like "처음부터 다시"). Recognize these patterns when they appear and respond per the catalog.

**Progress indicator (mandatory)**: at the start of every step, print a one-line Korean banner so the non-developer always knows where they are. Format:

```
🎯 [N/5] {친근한 한국어 단계 이름} · 예상 {시간}분
```

Concrete labels to use:
- Step 0 (only when a conflict exists): `🎯 [0/5] 기존 파일 확인 · 1분 이내`
- Step 1: `🎯 [1/5] 아이디어 입력 · 약 1–5분` (Quick 모드 1분, Template 모드 3–5분)
- Step 2: `🎯 [2/5] 빠진 정보 확인 질문 · 약 2–4분`
- Step 3: `🎯 [3/5] 자동화 설계서(docs/PRD.md) 만들기 · 약 2–3분`
- Step 4: `🎯 [4/5] 클로드 규칙 파일(CLAUDE.md) {+ n8n 워크플로우} 만들기 · 약 2–4분` (n8n 산출물은 build method가 n8n일 때만 추가 1~2분)
- Step 5: `🎯 [5/5] 완료 — /plan 으로 넘기기`

Skip the banner for Step 0 when no conflict is found (the step is silent in that case).

### Step 0: Pre-flight check (간단 확인)

Before showing the template, inspect the project root for existing artifacts so a re-run never silently destroys earlier work. Keep this step minimal — non-developers should not have to choose between "덮어쓰기 / 수정 모드 / 취소" trichotomies.

1. Use Read to check whether `docs/PRD.md` exists.
2. Use Read to check whether `CLAUDE.md` exists in the project root.
3. If **neither** file exists, continue silently to Step 1 — do not bother the user.
4. If **either** file exists, call AskUserQuestion exactly once with a single Y/N choice:
   - Question: "이미 `자동화 설계서(docs/PRD.md)` 또는 `클로드 규칙 파일(CLAUDE.md)` 이 있어요. 덮어쓸까요? (기존 파일은 자동으로 `.bak` 으로 백업해드려요)"
   - Options:
     - `네, 덮어써요 (추천)` — proceed with the steps below
     - `아니요, 취소할게요` — stop the skill and tell the user how to resume later (e.g., "기존 파일을 직접 옮기거나 지운 뒤 다시 `/hackathon-setup` 실행해주세요")
5. If the user picks `네, 덮어써요`, silently back up each existing file with Bash `cp` before any later Write (`docs/PRD.md` → `docs/PRD.md.bak`, `CLAUDE.md` → `CLAUDE.md.bak`). Do this now in Step 0 so later steps don't need to re-check. Do not mention `.bak` filenames during the conversation — the user only needs to know a backup exists if they ask.
6. There is no "수정 모드" branch. Re-running the skill always regenerates fresh drafts from Step 1. If the user wants to keep parts of the old file, they edit the file directly after `/hackathon-setup` finishes.

### Step 1: Entry mode + input collection

Before showing the template, give the user a choice of how to start. PRD 작성 경험이 없는 비개발자에겐 빈 6섹션 템플릿이 그 자체로 진입 장벽이 됨 → 짧게 시작할 수 있는 옵션을 먼저 제시.

#### Step 1a: Entry mode (필수)

Call AskUserQuestion: "어떻게 시작하시겠어요?" with options:
- `한두 문장만 말할게요 (Claude가 펼쳐드릴게요, 약 1분)` — Quick mode (Step 1b-quick로 진행). PRD 처음 써보는 비개발자 추천.
- `템플릿 채우기 (꼼꼼하게, 약 5분)` — Template mode (Step 1b-template로 진행). 이미 머릿속에 그림이 잡힌 사람 추천.
- `예시부터 볼래요` — Read `references/examples/example_prd.md` 핵심 1~2 섹션을 화면에 발췌해서 보여준 뒤, 다시 위 2개 옵션 중 하나로 라우팅.

#### Step 1b-quick: Quick mode (1~2문장 → Claude가 6섹션 추측)

1. Ask in plain Korean: "어떤 업무를 자동화하고 싶으세요? 한두 문장으로 자유롭게 말씀해주세요. (예: '매주 광고 리포트 만드는 게 너무 오래 걸려서 슬랙으로 자동 발송됐으면 좋겠어요')"
2. Receive a single free-text reply.
3. **Vagueness check (per failure mode F14)**: 만약 답변이 1줄 미만이거나 "자동화하고 싶어요" 수준의 일반론이면, 한 번만 더 묻기 — "(a) 어떤 업무가 (b) 얼마나 자주 (c) 어떤 결과물이 나오면 좋겠는지 한두 문장 더 부탁드려요." 두 번째 답변까지 너무 모호하면 Template mode로 폴백 (1b-template).
4. Read `references/input_template_ko.md` (for the 7-section anchors — 3 필수 + 4 선택 including `🛠️ 만들 방식`) and **draft a 7-section Korean guess** based on the user's 1~2 sentences plus reasonable domain inference. Mark every guessed bullet with `[추측: ...]`. For `🛠️ 만들 방식`: if the user mentioned a specific tool (예: "n8n으로", "노코드로", "코드로"), capture it; otherwise set to `추천받을게요` (default — Step 3 will recommend).
5. Show the draft on screen and ask in Korean: "이렇게 이해했는데 맞을까요? 틀린 부분이나 빠진 부분만 알려주세요. 그대로 좋으면 '맞아요' 라고 답해주세요."
6. Accept their corrections (free text), update the 6-section draft, then proceed to Step 2. Carry the `[추측: ...]` markers forward — Step 2 will resolve them along with normal gaps.

#### Step 1b-template: Template mode (현재 기본 흐름)

Read `references/input_template_ko.md` and show its 6 sections inline. Each section already includes one example bullet so the user has something to anchor on.

Tell the user once, in plain Korean:
- 어떤 섹션은 필수 (문제 / 원하는 것 / 사용 범위), 나머지는 선택이라는 점
- 모르는 항목은 비워두거나 "잘 모르겠어요" 라고 써도 된다는 점
- 답변은 한국어로 자유롭게 쓰면 된다는 점
- 특히 **문제 섹션의 "지금 손으로 하는 단계 3~5개"는 꼭 채워주세요** — 자동화 어디를 맡길지 결정하는 핵심 정보예요.
- **🛠️ 만들 방식 섹션은 잘 모르면 "추천받을게요"라고 쓰시면 돼요** — PRD 초안이 나오면 클로드가 두 가지 중 어떤 게 더 적합한지 추천해드려요.

Then collect the bullets. Prefer a single long-form text reply over six separate AskUserQuestion calls — fewer interaction beats keeps non-developer users from dropping out. If the user replies with only a sentence or two, treat the missing sections as "TBD" and let Step 2 fill them in.

### Step 2: Clarification loop (max 2 rounds × max 3 questions)

Analyze the Step 1 input and find the highest-impact gaps. Read `references/clarification_questions.md` for a curated pool, then adapt the wording to the user's specific situation.

**Round 1 (필수)**: 다음 2개 질문은 *Step 1에서 이미 명확히 답변되지 않았다면 반드시 포함* — 6시간 해커톤의 성패를 좌우하는 변수임.
1. **A4 (As-is 워크플로우 단계)** — 사용자가 지금 손으로 하는 단계 3~5개. 이게 없으면 자동화 진입점을 식별할 수 없음. Step 1에서 "단계가 잘 안 떠올라요" 라고 답했으면, 사용자의 Step 1 입력 + 도메인 상식으로 3~5단계 초안을 직접 작성해 보여주고 "이 흐름이 맞나요?"로 확인 받기.
2. **F1 (시스템 접근 권한)** — 자동화에 필요한 시스템 접근 권한 보유 여부. 답이 "모르겠어요" / "일부만"이면 PRD section 12 (Access Risk)에 자동으로 `[Risk: 접근권한 사전 확인 필요]` 마킹.

**Round 1 남은 1개 슬롯**: Step 1 답변에서 가장 큰 갭(보통 출력 destination, 빈도, hard constraint 중 하나)을 하나 더 골라 묻기. Call AskUserQuestion with one question at a time, each option carrying a concrete example answer (e.g., "주 1회 — 매주 월요일 오전").

After Round 1, check the data model: do you now have enough to write `purpose`, `target_user`, `as_is_workflow`, `access_status`, `input_type`, `output_type`, `output_destination`, `frequency`, `scope` ? If yes, skip Round 2 — the user pays a fatigue cost for every avoidable question.

Round 2 (only if needed): up to 3 more questions on remaining gaps. After Round 2, stop asking — fill unknown fields with `[가정: ...]` markers and proceed.

### Step 3: PRD Korean summary → English save

Print the progress banner `🎯 [3/5] 자동화 설계서(docs/PRD.md) 만들기 · 약 2–3분`.

**Reference reading (mandatory before drafting)**:
- Read `references/prd_template_en.md` for the section structure (14 sections including Section 0 Quick Facts & Hand-off, As-is workflow, MVP, Phase 2, Access Risk, Build Method).
- Read `references/examples/example_prd.md` for the quality bar — mimic the level of detail, not the domain.

Compose the PRD draft on screen in Korean using the 14-section structure. The principle here is **"draft maximally, ask minimally"** — Claude fills as much as it can from Step 1/2 data, then surfaces *one* unified confirmation. Most validation happens silently as part of drafting, not as separate AskUserQuestion gates.

#### Step 3a: Silent draft pass (no user-facing questions during this phase)

Before showing anything to the user, Claude works through the following inline. Each item is *baked into the draft*, not asked as a separate question.

1. **Required content sections** — `Problem` (with As-is workflow), `Target User`, `Goals & Success Metrics`, `Inputs & Data Sources`, `Output`. If any is empty after Step 2, fill with `[가정: 이 항목은 셋업 중에 결정 못 함]` directly in the draft. (Only escalate to a real AskUserQuestion in Step 3c if the user explicitly says "수정할게요".)

2. **MVP draft (Section 10)** — if the user gave an MVP in Step 1/2 use it; otherwise Claude proposes the smallest single-trigger / single-output version inferred from the PRD content, and marks it with `[추측: 클로드가 제안한 6시간 MVP — 다르면 수정 화면에서 알려주세요]`. Everything else from `In scope` moves to Section 11 (Phase 2). Never silently drop user wishes.

3. **Access Risk (Section 12)** — copy directly from Step 2 question F1 answer. Auto-set Section 0's `Access risk` field: `none` (모두 있음) / `partial` (일부만) / `unknown — must verify first` (모름).

4. **Build Method (Section 13)** — two sub-branches:
   - **Sub-branch A — Step 1에서 이미 `Claude Code` 또는 `n8n`을 명시한 경우**: Use it silently. No AskUserQuestion. Section 13's "Selected" field reflects the choice, "Why this fits" is one Claude-generated line based on PRD content. The surfaced confirmation (Step 3c) will say "Build method: {선택} — Step 1에서 선택하신 대로" in the summary so the user can override there if needed.
   - **Sub-branch B — Step 1에서 `추천받을게요` 또는 빈칸/모름**: Claude picks based on heuristic:
     - 외부 API 3개 이상 + 데이터 변환이 단순 → **n8n**
     - 커스텀 로직/AI 호출/복잡한 데이터 가공이 핵심 → **Claude Code**
     - n8n 인스턴스 보유 "아니요" 또는 "모르겠어요" → **Claude Code**
     - n8n 인스턴스 보유 "네" + API 통합 중심 → **n8n 강화**
   - The surfaced confirmation (Step 3c) will say "Build method: **{추천} (클로드 추천)** — {이유 1줄}. 다른 방식으로 하려면 수정 화면에서 알려주세요." Single line, no separate AskUserQuestion.

5. **Section 0 (Quick Facts & Hand-off) 자동 합성** — Build method / Hackathon MVP / Access risk / Category tag (Claude가 PRD 도메인 보고 추론: marketing-report, hr-onboarding, cs-triage, sales-followup, ops-monitoring, etc.) + Hand-off rules block (per `prd_template_en.md` Section 0). Fill all lines automatically; no separate user question.

6. **`[가정: ...]` 카운트** — count assumption markers in the draft. If ≥ 5, *fold the count into the confirmation prompt text* (Step 3c) rather than triggering a separate AskUserQuestion. The unified confirmation will say "가정으로 채운 항목이 N개예요. 그대로 저장할까요, 수정할까요?"

#### Step 3b: (no separate step — silent pass continues into 3c)

#### Step 3c: Single unified confirmation (★ the ONE user-facing question in Step 3)

Print the full PRD draft in Korean on screen, immediately followed by a 3-line summary footer:

```
─────
요약:
• 만들 방식: {Claude Code | n8n} {— Step 1 선택 / — 클로드 추천 (이유 1줄)}
• 6시간 MVP: {한 줄 요약} {— Step 1 입력 / — 클로드 제안 (수정 가능)}
• 가정 항목: N개 {(많으면 /plan에서 추가로 다듬어드려요)}
─────
```

Then ONE AskUserQuestion: "이대로 자동화 설계서(docs/PRD.md)로 저장할까요?" with options:
- `네, 저장할게요 (추천)`
- `수정할게요` — accept free-text edits (the user can mention any section: build method / MVP / 다른 섹션). Re-draft inline, re-show, re-confirm.
- `취소`

**Confirmation loop budget**: track `confirmation_round_count`. Each `수정할게요` increments. After 3 rounds switch to F4 wording (recommend save + edit later). Hard cap at 5 rounds — past that save regardless.

**Edge case — extreme assumption count or user dissatisfaction**: if the user picks `수정할게요` *and* the assumption count is ≥ 5 *and* the user's edits don't address Step 2's known gaps, ask once whether they want to loop back to Step 2 for one extra clarification round (this is the only exception to Step 2's "max 2 rounds" cap, and it's user-initiated via F5 in `failure_modes.md`).

#### Step 3d: Save

On `네, 저장할게요`:
1. Use Bash `mkdir -p docs` to guarantee the directory exists.
2. Translate the confirmed summary into English using the structure in `references/prd_template_en.md`. Preserve every `[Assumption: ...]` tag (translated from `[가정: ...]`) and `[추측: ...]` markers (translate to `[Guess: ...]`).
3. Use Write to save to `docs/PRD.md`.
4. **Store build method** in working memory (Step 4d uses this to decide whether to generate `docs/n8n-workflow.md`).
5. Report in one short Korean line: "저장 완료: docs/PRD.md".

### Step 4: CLAUDE.md Korean preview → English save

#### Step 4a: Collect API docs / guide links

Before generating the CLAUDE.md draft, ask the user for any external API documentation, guides, or reference links they want Claude to remember throughout the project. These will be embedded verbatim in the CLAUDE.md `## References` section so every future Claude Code session has them in context.

Call AskUserQuestion: "프로젝트에서 참고할 API 문서나 가이드 링크가 있나요? (예: Slack API 문서, Google Sheets API 가이드 등) 클로드 규칙 파일의 ## References 섹션에 그대로 저장해서 매번 참고할 수 있게 해드려요." with options:
- `네, 링크 있어요` — 사용자가 자유 텍스트로 URL 1개 이상 입력. Other 입력을 받아 줄바꿈/쉼표로 구분된 링크를 모두 파싱.
- `아니요, 없어요` — `## References` 섹션은 `docs/PRD.md` 1개만 남기고 건너뜀.
- `잘 모르겠어요` — 동일하게 빈 References로 진행하되, "/plan 실행 후 필요하면 직접 추가할 수 있어요" 라고 한 줄 안내.

Store the collected links (URL + optional 짧은 설명) for use in Step 4b's draft.

#### Step 4b: Korean preview (slim — 3 sections only)

**Reference reading (mandatory before drafting)**:
- Read `references/claude_md_template_en.md` to anchor the structure. **The template is intentionally slim: 3 sections only.**
- Read `references/examples/example_claude_md.md` for the quality bar — a slim CLAUDE.md (< 30 content lines) that mimics specificity through pointers to the PRD, not duplication.

Generate a CLAUDE.md draft in Korean for on-screen review. **3 sections only**:
1. **Project Overview** — 2~4 sentences. What this project does, who runs it, why it exists.
2. **Working Style** — the rules that change every Claude response in this project: conversation language (Korean), file language (English), non-developer default, credential-verification rule, ambiguity handling, change-summarization. 6 bullets max.
3. **References** — `docs/PRD.md` first (점토 of truth pointer), then any external URLs from Step 4a verbatim.

**Do NOT include**: Tech Constraints, Success Criteria, Out of Scope, Primary User details, Hand-off Notes. All of these now live in `docs/PRD.md` Section 0 (Quick Facts & Hand-off). Duplicating them here creates drift.

**Quality validation gates (run silently BEFORE the confirmation prompt)**:

1. **3-section rule**: exactly the sections `Project Overview` / `Working Style` / `References`. If the draft has any other top-level section, remove it and *fold the content into the PRD instead* (which is editable after the run).
2. **References parity**: `## References` section must contain at least `docs/PRD.md`. If missing, restore it.
3. **Line budget (★ hard cap)**: total file length MUST be ≤ 50 lines including headings and blank lines. If the draft exceeds 50 lines, compress until it fits — do NOT ask the user. CLAUDE.md is auto-loaded into every future session in this project, so token economy matters.
4. **No content duplication with PRD**: if any bullet in `Working Style` restates a PRD Section 0 Hand-off rule verbatim, that's fine (parallel reinforcement). If it restates Tech Constraints / Success Criteria / Out of Scope content, *delete that bullet* — PRD owns that content.

#### Step 4c: Confirm and save

Call AskUserQuestion: "이 내용으로 클로드 규칙 파일(CLAUDE.md)을 저장할까요?" with options `네, 저장해요 (추천)`, `수정할게요`, `취소`. Accept free-text edits if requested and re-show. If the user wants to add/remove links during the edit step, treat that as a valid edit and update the `## References` section before re-showing. Apply the same `confirmation_round_count` cap (3 rounds → F4 wording, 5 rounds → save regardless) as Step 3.

On confirmation:
1. Translate the confirmed Korean draft to English using `references/claude_md_template_en.md`. The `## References` section is bilingual-safe — URLs are preserved as-is; only the surrounding descriptions are translated.
2. Use Write to save to `CLAUDE.md` at project root. (Backup of any prior `CLAUDE.md` was already made in Step 0, so no extra backup logic is needed here.)
3. Report the absolute path.

#### Step 4d: (조건부) n8n 워크플로우 산출물 생성

**Trigger condition**: Step 3에서 결정된 build method가 `n8n`인 경우에만 실행. 그 외(`Claude Code`)면 이 단계 전체를 skip하고 Step 5로 진행.

**Reference reading (mandatory)**:
- Read `references/n8n_workflow_template_en.md` for the node-graph structure.
- Read `references/examples/example_n8n_workflow.md` for the quality bar — 5-node MVP example. Mimic the level of node-type specificity (real n8n node names, real field configs), not the domain content.

Generate a Korean preview of the n8n workflow draft using the template structure. Sections to fill:
- Trigger (type + schedule/event + 1줄 근거)
- Node sequence (3~7 노드, 각 노드별 Node type / Purpose / Key config / Output)
- Branches & Error handling (MVP minimum — happy path + empty-data fallback)
- Required Credentials (n8n Credentials store 기준; PRD section 12 Access Risk 와 일치시켜야 함)
- Estimated complexity (node count / API 수 / 빌드 예상 시간 / 난이도)
- What this does NOT cover (PRD section 11 Phase 2 verbatim 복사)
- Visual reference (선택 — ASCII 체인 1줄)

**Quality validation gate**:
1. **Node count guard**: 노드 8개 이상이면 MVP 스코프 폭주 가능성 → 사용자에게 "노드가 N개네요. MVP에 너무 많을 수 있어요. Phase 2로 옮길 만한 게 있을까요?" 1회 확인.
2. **Credential parity**: PRD section 12에서 `needs request` 마킹된 dependency는 n8n credentials 표에도 동일하게 `needs request`로 반영. 자동 동기화.
3. **Node type 정확성**: 모든 node type이 실제 n8n에 존재하는 이름인지 확인 (Schedule Trigger / Webhook / HTTP Request / Set / IF / Switch / Code / Function / Slack / Google Sheets / Notion / ...). 가공의 노드 이름 금지.

Call AskUserQuestion: "이 n8n 워크플로우 설계서를 저장할까요?" with options `네, 저장해요 (추천)`, `수정할게요`, `취소`. Apply the same `confirmation_round_count` cap as Steps 3/4c.

On confirmation:
1. Translate the confirmed Korean draft to English using `references/n8n_workflow_template_en.md`.
2. Use Write to save to `docs/n8n-workflow.md`.
3. Report the absolute path in one short Korean line: "저장 완료: docs/n8n-workflow.md".

### Step 5: Handoff to /plan (수동 트리거)

Output a short Korean closing message. **Build method에 따라 생성된 파일 목록과 권장 프롬프트가 다름** — 아래 두 분기 중 해당하는 것을 선택해서 출력.

**분기 A: Build method = Claude Code**

```
셋업이 끝났어요! 🎉

생성된 파일:
- 자동화 설계서: docs/PRD.md
- 클로드 규칙 파일: CLAUDE.md

👉 다음으로 할 일 (순서 중요!)

1. CLAUDE.md 를 열어서 내용을 한 번 훑어봐 주세요.
   - 잘못 적힌 규칙이 있으면 지금 직접 고쳐도 돼요.
   - 이 파일은 앞으로 모든 클로드 세션에서 자동으로 읽혀요.

2. 내용이 마음에 들면, 아래 프롬프트를 그대로 복사해서 입력해 주세요:

     /plan docs/PRD.md 를 읽고 단계별 실행 계획 세워줘. docs/PRD.md Section 0 의 Hand-off 규칙을 반드시 지켜줘.

   이 프롬프트는 PRD 위치(docs/PRD.md)와 따라야 할 규칙(PRD Section 0 의 Hand-off block)을 /plan 에 한 번에 전달해줘서,
   /plan 이 다시 묻지 않고 바로 계획 작성에 들어갑니다.
   계획을 검토한 뒤 마음에 들면 그대로 코드 생성까지 이어집니다.
```

**분기 B: Build method = n8n**

```
셋업이 끝났어요! 🎉

생성된 파일:
- 자동화 설계서: docs/PRD.md
- n8n 워크플로우 설계서: docs/n8n-workflow.md
- 클로드 규칙 파일: CLAUDE.md

👉 다음으로 할 일 (순서 중요!)

1. CLAUDE.md 와 docs/n8n-workflow.md 를 한 번씩 훑어봐 주세요.
   - 잘못 적힌 노드/필드가 있으면 지금 직접 고쳐도 돼요.
   - PRD section 12 (Access Risk) 가 `needs request` 인 자격증명이 있으면 먼저 발급 요청 들어가세요 — 시간이 가장 오래 걸리는 항목이에요.

2. n8n 인스턴스에 워크플로우를 만들 때:
   - docs/n8n-workflow.md 의 노드 순서 그대로 드래그앤드롭하면 됩니다.
   - 각 노드의 Key config 섹션을 그대로 복붙해서 채우세요.
   - Required Credentials 표를 먼저 만들어두고 시작하세요.

3. 막히거나 클로드와 추가 논의가 필요하면, 아래 프롬프트를 복사해서 입력해 주세요:

     /plan docs/PRD.md 와 docs/n8n-workflow.md 를 읽고, n8n 인스턴스에서 단계별로 구현할 가이드를 만들어줘. docs/PRD.md Section 0 의 Hand-off 규칙을 반드시 지켜줘.

   이 프롬프트는 PRD + n8n 설계 + 규칙 3개를 한 번에 전달해줘서, /plan 이 노드별 빌드 가이드를 작성해줍니다.
```

**Critical rule — do NOT auto-trigger `/plan`.**

- Reason: non-developers must read `CLAUDE.md` before development starts. The file becomes a persistent rule set for every future Claude Code session in this project, and silent mistakes (wrong scope, wrong constraints, wrong success criteria) propagate everywhere downstream. Skipping the review step is a high-risk shortcut for this audience.
- Implementation: end the skill after printing the closing message. Do not call `Skill(skill="plan")`, do not run `/plan` via Bash, do not chain into any planner agent. The user types `/plan` themselves when they are ready.
- If the user replies "그냥 바로 plan 돌려줘" or similar, still refuse the auto-trigger and explain in one Korean sentence that CLAUDE.md 검토는 1~2분이면 끝나고, 그 뒤에 직접 `/plan` 을 입력해야 안전하다고 안내한다.
- The recommended `/plan` prompt printed in the closing message is for the USER to paste — do NOT issue it yourself via Skill/Bash/agent chaining. Print it as plain text in the closing message and stop.

## References

- **`references/input_template_ko.md`** — 6-section Korean template (필수/선택 그룹화) with example bullets, shown to user in Step 1b-template.
- **`references/prd_template_en.md`** — English PRD structure with 14 sections (Section 0 Quick Facts & Hand-off — single source of truth for `/plan` and downstream agents — plus Problem with As-is workflow, MVP, Phase 2, Access Risk, Build Method, Assumptions, ...) used in Step 3.
- **`references/claude_md_template_en.md`** — Slim English CLAUDE.md structure (3 sections only: Project Overview / Working Style / References, ≤50 lines hard cap). All Hand-off / Tech Constraints / Success Criteria / Out of Scope content now lives in PRD Section 0 — CLAUDE.md just points to it.
- **`references/n8n_workflow_template_en.md`** — English n8n workflow doc structure (Trigger, Node sequence, Branches, Credentials, Complexity). Used in Step 4d (only when build method = n8n).
- **`references/clarification_questions.md`** — Pool of clarifying questions grouped by topic (A=Problem, B=Output, C=Scope, D=Data, E=Constraints, F=Feasibility). Round 1 필수 항목 명시. Source for Step 2.
- **`references/examples/example_prd.md`** — Fully-completed PRD (Weekly Marketing Summary Bot) showing all 14 sections including MVP/Phase 2/Access Risk/Build Method. Quality bar for Step 3 — mimic the specificity, not the domain.
- **`references/examples/example_claude_md.md`** — Paired CLAUDE.md for the same example. Quality bar for Step 4.
- **`references/examples/example_n8n_workflow.md`** — 5-node n8n MVP example paired with the marketing bot PRD. Quality bar for Step 4d.
- **`references/failure_modes.md`** — Catalog of non-happy-path scenarios (F1–F15) covering conversational drift, content quality risks, quick-mode vagueness (F14), build-method ambiguity (F15), mechanical failures, and recovery commands. Read once at the start of every run.

## Recovery Commands

The user may issue these utterances at any point in the conversation. Detect them by intent (not exact string match) and route to the indicated handler. Full handler logic lives in `references/failure_modes.md` (F11–F13).

| User utterance (Korean intent) | Handler | Result |
|-------------------------------|---------|--------|
| "처음부터 다시", "처음부터 시작", "리셋", "다시 시작" | F11 | End the run, no files saved beyond what was already written. Ask user to re-trigger `/hackathon-setup`. |
| "이전 단계로", "Step N로 돌아가요", "방금 답 취소", "되돌리기" | F12 | This skill is strictly linear. Explain in one Korean sentence and offer: (a) re-trigger `/hackathon-setup` for a clean restart, or (b) edit the saved file directly after the run. |
| "취소", "그만", "stop", "중단" | F13 | Exit immediately. List any files already saved or backed up so the user knows the on-disk state. |
| "왜 자꾸 영어로 저장돼?" / "한국어로 저장해줘" | Policy explainer | Explain in one line: "클로드가 영어 스펙을 더 정확하게 따라가서 그래요. 저장된 후 직접 한국어로 바꾸셔도 됩니다." Do NOT switch the file language — the policy is fixed. |
| "/plan 자동으로 돌려줘" / "그냥 바로 진행" | Step 5 critical rule | Refuse the auto-trigger per Step 5's `Critical rule` block. Explain CLAUDE.md 검토는 1~2분이면 끝난다고 안내. |

Do not invent additional recovery commands — anything outside this table should fall through to normal conversation handling.

## Token Budget

This skill loads the following into context, in order:

| When | File | Approx. size |
|------|------|-------------:|
| Session start (always, due to skills auto-load) | `SKILL.md` (frontmatter + ~360 lines) | ~2.7k tokens |
| Step 0 (always) | `references/failure_modes.md` (F1–F15) | ~2.2k tokens |
| Step 1 — Template mode only | `references/input_template_ko.md` | ~1.0k tokens |
| Step 1 — Quick mode only | (no extra file load — Claude drafts directly) | ~0k tokens |
| Step 2 (always) | `references/clarification_questions.md` | ~1.5k tokens |
| Step 3 (always) | `references/prd_template_en.md` + `references/examples/example_prd.md` | ~3.5k tokens |
| Step 4 (always) | `references/claude_md_template_en.md` + `references/examples/example_claude_md.md` (slim, both ≤50 lines) | ~1.0k tokens |
| Step 4d — only when build method = n8n | `references/n8n_workflow_template_en.md` + `references/examples/example_n8n_workflow.md` (Store Review based) | ~3.6k tokens |
| **Full-run total — Claude Code build path** | | **~11.9k tokens** |
| **Full-run total — n8n build path** | | **~15.5k tokens** |

Slim CLAUDE.md change (50-line cap + Hand-off moved to PRD) saves ~1.4k tokens in Step 4 *and* shrinks every future Claude Code session in the same project (since CLAUDE.md is auto-loaded at every session start downstream).

Numbers are approximate (English averages ~4 chars/token, Korean ~2 chars/token). Reference these when deciding whether to add new files — adding a 5k-token file would meaningfully shift this skill's footprint. Quick mode in Step 1 saves ~0.9k tokens compared to Template mode.

## Settings

| Setting | Default | Override |
|---------|---------|----------|
| Save location for PRD | `docs/PRD.md` | User can specify alternative path during Step 3 confirmation |
| Save location for CLAUDE.md | `CLAUDE.md` (project root) | Fixed — Claude Code requires this exact path |
| Max clarification rounds | 2 | Hard cap to protect non-developer attention budget |
| Max questions per round | 3 | Hard cap — same reason |
| Display language | Korean | All on-screen interaction stays Korean |
| File language | English | All saved files in English; explained to user once in Step 3 |
| Minimum Claude Code version | see `references/failure_modes.md` F10 | F10 documents the UX degradation pattern on older clients; update the version floor there as the upstream client changes |
