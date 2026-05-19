---
name: hackathon-setup
description: Guides a non-developer through a 5-step Korean conversation that produces docs/PRD.md and CLAUDE.md, ready to hand off to /plan. Use for internal Claude Code mini-hackathon participants who need help turning a work-automation idea into a structured spec.
when_to_use: Invoked only by the user typing `/hackathon-setup` (this skill has `disable-model-invocation: true`, so Claude never auto-invokes it). The user should type that command when they want to set up a hackathon project, draft an automation PRD, or have Claude help structure a work problem they don't know how to brief — situations they might describe as "사내 해커톤 셋업", "해커톤 시작", "프로젝트 셋업 도와줘", "아이디어 정리해줘", "PRD 만들어줘", "자동화 설계서", or "hackathon setup". If the user describes such a situation in natural language without typing the slash command, suggest they run `/hackathon-setup` themselves rather than auto-launching the workflow.
disable-model-invocation: true
---

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
- Step 1: `🎯 [1/5] 아이디어 입력 · 약 3–5분`
- Step 2: `🎯 [2/5] 빠진 정보 확인 질문 · 약 2–4분`
- Step 3: `🎯 [3/5] 자동화 설계서(docs/PRD.md) 만들기 · 약 2–3분`
- Step 4: `🎯 [4/5] 클로드 규칙 파일(CLAUDE.md) 만들기 · 약 2–3분`
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

### Step 1: Template guidance + input collection

Read `references/input_template_ko.md` and show its 6 sections inline. Each section already includes one example bullet so the user has something to anchor on.

Tell the user once, in plain Korean:
- 어떤 섹션은 필수 (문제 / 원하는 것 / 사용 범위), 나머지는 선택이라는 점
- 모르는 항목은 비워두거나 "잘 모르겠어요" 라고 써도 된다는 점
- 답변은 한국어로 자유롭게 쓰면 된다는 점

Then collect the bullets. Prefer a single long-form text reply over six separate AskUserQuestion calls — fewer interaction beats keeps non-developer users from dropping out. If the user replies with only a sentence or two, treat the missing sections as "TBD" and let Step 2 fill them in.

### Step 2: Clarification loop (max 2 rounds × max 3 questions)

Analyze the Step 1 input and find the highest-impact gaps. Read `references/clarification_questions.md` for a curated pool, then adapt the wording to the user's specific situation.

Round 1: Pick up to 3 of the most critical missing pieces (typically: real frequency of the problem, where the output lives, who else uses the result). Call AskUserQuestion with one question at a time, each option carrying a concrete example answer (e.g., "주 1회 — 매주 월요일 오전").

After Round 1, check the data model: do you now have enough to write `purpose`, `target_user`, `input_type`, `output_type`, `output_destination`, `frequency`, `scope` ? If yes, skip Round 2 — the user pays a fatigue cost for every avoidable question.

Round 2 (only if needed): up to 3 more questions on remaining gaps. After Round 2, stop asking — fill unknown fields with `[가정: ...]` markers and proceed.

### Step 3: PRD Korean summary → English save

Print the progress banner `🎯 [3/5] 자동화 설계서(docs/PRD.md) 만들기 · 약 2–3분`.

**Reference reading (mandatory before drafting)**:
- Read `references/prd_template_en.md` for the section structure.
- Read `references/examples/example_prd.md` for the quality bar — a fully-completed PRD that shows the level of specificity, concreteness, and out-of-scope discipline to aim for. Mimic the level of detail, not the domain content.

Compose a 10-section Korean summary on screen using that structure. Surface every assumption as `[가정: ...]` so the user can spot guesses. Use the friendly vocabulary mapping when naming the file.

**Quality validation gate (run BEFORE the confirmation prompt)**:

1. Check that all required sections have at least one non-empty bullet: `Problem`, `Target User`, `Goals & Success Metrics`, `Inputs & Data Sources`, `Output`. If any of these is empty or TBD-only:
   - Run ONE more targeted AskUserQuestion for the missing section's most critical sub-bullet (per failure mode F6 in `references/failure_modes.md`).
   - If still empty after that single retry, save with `[가정: 이 항목은 셋업 중에 결정 못 함]` and continue.
2. Count the number of `[가정: ...]` markers in the draft. If **5 or more**:
   - Surface the count in Korean before showing the confirmation prompt: "가정으로 채운 항목이 N개예요. 더 답해주실래요, 아니면 가정 그대로 저장하고 `/plan` 단계에서 다듬을까요?"
   - Call AskUserQuestion with two options: `더 답할게요 (확인 질문 한 라운드 더)` / `이대로 저장 (가정은 /plan에서 다듬기, 추천)`. If user picks the first, loop back to Step 2 for one extra round (this is the only exception to the Step 2 "max 2 rounds" cap, and it's user-initiated).
3. Track a `confirmation_round_count`. Each time the user picks `수정할게요` in the next step, increment it. After 3 rounds, switch the confirmation prompt to the F4 wording in `references/failure_modes.md`. Hard cap at 5 rounds — past that, save and move on regardless.

Then call AskUserQuestion: "이대로 자동화 설계서(docs/PRD.md)로 저장할까요?" with options `네, 저장해요 (추천)`, `수정할게요`, `취소`. If `수정할게요`, accept their edits in free text, regenerate the Korean summary, and re-confirm.

On confirmation:
1. Use Bash `mkdir -p docs` to guarantee the directory exists.
2. Translate the confirmed summary into English using the structure in `references/prd_template_en.md`. Preserve every `[Assumption: ...]` tag (translated from `[가정: ...]`).
3. Use Write to save to `docs/PRD.md`.
4. Report the absolute path in one short Korean line: "저장 완료: docs/PRD.md".

### Step 4: CLAUDE.md Korean preview → English save

#### Step 4a: Collect API docs / guide links

Before generating the CLAUDE.md draft, ask the user for any external API documentation, guides, or reference links they want Claude to remember throughout the project. These will be embedded verbatim in the CLAUDE.md `## References` section so every future Claude Code session has them in context.

Call AskUserQuestion: "프로젝트에서 참고할 API 문서나 가이드 링크가 있나요? (예: Slack API 문서, Google Sheets API 가이드 등) 클로드 규칙 파일의 ## References 섹션에 그대로 저장해서 매번 참고할 수 있게 해드려요." with options:
- `네, 링크 있어요` — 사용자가 자유 텍스트로 URL 1개 이상 입력. Other 입력을 받아 줄바꿈/쉼표로 구분된 링크를 모두 파싱.
- `아니요, 없어요` — `## References` 섹션은 `docs/PRD.md` 1개만 남기고 건너뜀.
- `잘 모르겠어요` — 동일하게 빈 References로 진행하되, "/plan 실행 후 필요하면 직접 추가할 수 있어요" 라고 한 줄 안내.

Store the collected links (URL + optional 짧은 설명) for use in Step 4b's draft.

#### Step 4b: Korean preview

**Reference reading (mandatory before drafting)**:
- Read `references/claude_md_template_en.md` to anchor the structure.
- Read `references/examples/example_claude_md.md` for the quality bar — a fully-completed CLAUDE.md paired with the example PRD from Step 3. Mimic the level of specificity (testable success criteria, explicit out-of-scope bullets, non-developer language), not the domain content.

Generate a CLAUDE.md draft in Korean for on-screen review only — it must cover: project overview, primary user, tech constraints (often "non-developer, prefer no-code or single-script solutions"), success criteria, things explicitly out of scope, working style notes (including the Korean conversation / English file policy), and a `## References` section that lists `docs/PRD.md` plus every link collected in Step 4a (verbatim URLs).

**Quality validation gate (run BEFORE the confirmation prompt)**:

1. `## References` section must contain at least one entry — `docs/PRD.md`. If empty, restore it before showing the preview.
2. If the PRD's `Target User` mentioned "non-developer" (or the user self-identified as such in Step 1/2), the CLAUDE.md `Tech Constraints` section MUST explicitly include a non-developer ownership bullet. If missing, add it before showing the preview.
3. **`## Hand-off Notes for /plan and Downstream Agents` section is mandatory.** This is the inter-skill handoff payload that `/plan` and every downstream Claude Code session reads automatically. It MUST contain at minimum: (a) user technical level (non-developer vs developer), (b) language policy (e.g., "explain in Korean, code in English"), (c) credential-verification rule (do not invent setup steps), (d) the "done" quality bar pointer back to Success Criteria. See `references/examples/example_claude_md.md` for the exact structure. If the draft omits this section, add it before showing the preview.
4. Total file length should be under 200 lines (Anthropic's official CLAUDE.md guidance) and ideally under 80 lines for hackathon scope. If the draft exceeds 200 lines, trim verbose prose to bullet form before showing the preview — do NOT ask the user to do this.

#### Step 4c: Confirm and save

Call AskUserQuestion: "이 내용으로 클로드 규칙 파일(CLAUDE.md)을 저장할까요?" with options `네, 저장해요 (추천)`, `수정할게요`, `취소`. Accept free-text edits if requested and re-show. If the user wants to add/remove links during the edit step, treat that as a valid edit and update the `## References` section before re-showing. Apply the same `confirmation_round_count` cap (3 rounds → F4 wording, 5 rounds → save regardless) as Step 3.

On confirmation:
1. Translate the confirmed Korean draft to English using `references/claude_md_template_en.md`. The `## References` section is bilingual-safe — URLs are preserved as-is; only the surrounding descriptions are translated.
2. Use Write to save to `CLAUDE.md` at project root. (Backup of any prior `CLAUDE.md` was already made in Step 0, so no extra backup logic is needed here.)
3. Report the absolute path.

### Step 5: Handoff to /plan (수동 트리거)

Output a short Korean closing message:

```
셋업이 끝났어요! 🎉

생성된 파일:
- 자동화 설계서: docs/PRD.md
- 클로드 규칙 파일: CLAUDE.md

👉 다음으로 할 일 (순서 중요!)

1. CLAUDE.md 를 열어서 내용을 한 번 훑어봐 주세요.
   - 잘못 적힌 규칙이 있으면 지금 직접 고쳐도 돼요.
   - 이 파일은 앞으로 모든 클로드 세션에서 자동으로 읽혀요.

2. 내용이 마음에 들면, 직접 아래 커맨드를 입력해 주세요:

     /plan

   /plan 은 PRD를 읽고 실행 계획을 세워줘요.
   계획을 검토한 뒤 마음에 들면 그대로 코드 생성까지 이어집니다.
```

**Critical rule — do NOT auto-trigger `/plan`.**

- Reason: non-developers must read `CLAUDE.md` before development starts. The file becomes a persistent rule set for every future Claude Code session in this project, and silent mistakes (wrong scope, wrong constraints, wrong success criteria) propagate everywhere downstream. Skipping the review step is a high-risk shortcut for this audience.
- Implementation: end the skill after printing the closing message. Do not call `Skill(skill="plan")`, do not run `/plan` via Bash, do not chain into any planner agent. The user types `/plan` themselves when they are ready.
- If the user replies "그냥 바로 plan 돌려줘" or similar, still refuse the auto-trigger and explain in one Korean sentence that CLAUDE.md 검토는 1~2분이면 끝나고, 그 뒤에 직접 `/plan` 을 입력해야 안전하다고 안내한다.

## References

- **`references/input_template_ko.md`** — 6-section Korean template with example bullets per section, shown to user in Step 1.
- **`references/prd_template_en.md`** — English PRD structure (Problem, Goals, Users, Scope, Constraints, Success Metrics, Assumptions) used in Step 3.
- **`references/claude_md_template_en.md`** — English CLAUDE.md structure (Project Overview, Primary User, Tech Constraints, Success Criteria, Out of Scope) used in Step 4.
- **`references/clarification_questions.md`** — Pool of clarifying questions grouped by topic, source for Step 2.
- **`references/examples/example_prd.md`** — Fully-completed PRD (Weekly Marketing Summary Bot). Quality bar for Step 3 — mimic the specificity, not the domain.
- **`references/examples/example_claude_md.md`** — Paired CLAUDE.md for the same example. Quality bar for Step 4.
- **`references/failure_modes.md`** — Catalog of non-happy-path scenarios (F1–F13) covering conversational drift, content quality risks, mechanical failures, and recovery commands. Read once at the start of every run.

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
| Session start (always, due to skills auto-load) | `SKILL.md` (frontmatter + first ~200 lines) | ~1.5k tokens |
| Step 0 (always) | `references/failure_modes.md` | ~1.8k tokens |
| Step 1 (always) | `references/input_template_ko.md` | ~0.7k tokens |
| Step 2 (always) | `references/clarification_questions.md` | ~1.1k tokens |
| Step 3 (always) | `references/prd_template_en.md` + `references/examples/example_prd.md` | ~2.7k tokens |
| Step 4 (always) | `references/claude_md_template_en.md` + `references/examples/example_claude_md.md` | ~2.4k tokens |
| **Full-run total** | | **~10k tokens** |

Numbers are approximate (English averages ~4 chars/token, Korean ~2 chars/token). Reference these when deciding whether to add new files — adding a 5k-token file would meaningfully shift this skill's footprint.

## Settings

| Setting | Default | Override |
|---------|---------|----------|
| Save location for PRD | `docs/PRD.md` | User can specify alternative path during Step 3 confirmation |
| Save location for CLAUDE.md | `CLAUDE.md` (project root) | Fixed — Claude Code requires this exact path |
| Max clarification rounds | 2 | Hard cap to protect non-developer attention budget |
| Max questions per round | 3 | Hard cap — same reason |
| Display language | Korean | All on-screen interaction stays Korean |
| File language | English | All saved files in English; explained to user once in Step 3 |
| Minimum Claude Code version | `v2.1.141` | AskUserQuestion popup hiding bug fixed in this version; older versions degrade the non-developer UX |
