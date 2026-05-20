# Failure Modes & Edge Case Handling

> Catalog of non-happy-path scenarios this skill must handle gracefully. Read this once at the start of a `/hackathon-setup` run so you recognize the patterns when they occur. Each entry: **Trigger → Response → Why**.

## Conversational drift

### F1. User answers Step 1 with one sentence
- **Trigger**: "그냥 매일 슬랙으로 뭐 좀 보냈으면 좋겠어요" and nothing else.
- **Response**: Do NOT push back asking for all 6 sections. Acknowledge their answer in Korean, mark the 5 missing sections as `[가정: ...]` candidates, and proceed to Step 2 where you'll ask the highest-impact 3 clarification questions.
- **Why**: Non-developers anchor on the part they care about. Forcing them back to fill a template kills the conversation. Step 2 is the recovery mechanism.

### F2. User abandons Korean and switches to English mid-conversation
- **Trigger**: User types "actually let me explain in english because it's faster".
- **Response**: Continue accepting their input in English, but keep YOUR replies in Korean. Mention once: "영어로 답변 주셔도 괜찮아요 — 저는 계속 한국어로 안내드릴게요." Files still save in English (no policy change).
- **Why**: Forcing language consistency creates friction. The skill's contract is "display language = Korean", which means your side, not theirs.

### F3. User goes off on a tangent during Step 2
- **Trigger**: While answering "어디로 결과물이 전달되나요?", user starts describing a different automation idea.
- **Response**: Gently anchor: "지금은 [원래 자동화 이름] 셋업 중이에요. 새 아이디어는 메모해두고, 이번 셋업이 끝나면 `/hackathon-setup` 다시 실행해서 따로 정리해드릴게요." Do NOT mix two projects into one PRD.
- **Why**: Mixed-scope PRDs break `/plan`. One project per skill run is a hard rule.

### F4. User keeps picking "수정할게요" in Step 3 or Step 4 (loop > 3 times)
- **Trigger**: After 3 confirmation rounds, user still wants edits.
- **Response**: After the 3rd round, call AskUserQuestion with: "수정이 많네요. 지금까지 모은 내용을 그대로 저장하고 나중에 직접 편집하실래요, 아니면 한 번 더 수정하실래요?" Two options only: `지금 저장하고 직접 편집` (recommended) / `한 번 더 수정`. Cap at 5 rounds total — past that, save and move on regardless.
- **Why**: Confirmation loops are a non-developer trap. They feel safer iterating in chat than editing a saved file, but the result is decision fatigue.

## Content quality risks

### F5. `[가정: ...]` markers exceed 5 in the final PRD draft
- **Trigger**: Quality gate count (see Step 3 validation) returns ≥ 5.
- **Response**: Before showing the confirmation prompt, surface the count to the user in Korean: "가정으로 채운 항목이 N개예요. 더 답해주실래요, 아니면 가정 그대로 저장하고 `/plan` 단계에서 다시 다듬을까요?" Two options: `더 답할게요 (Step 2 추가 라운드)` / `이대로 저장 (가정은 /plan에서 다듬기)`.
- **Why**: 5+ assumptions usually means the PRD is too thin for `/plan` to act on. Catching it now is cheaper than catching it in implementation.

### F6. Required PRD section is empty after Step 2
- **Trigger**: One of `Problem` / `Target User` / `Goals` / `Inputs` / `Output` is still TBD after Step 2 closes.
- **Response**: Do NOT save the PRD silently with an empty section. Run ONE more targeted AskUserQuestion for the missing section's most critical sub-bullet. If still empty after that, save with `[Assumption: section deferred — user could not specify during setup]` and proceed.
- **Why**: Empty required sections make `/plan` produce garbage. One extra question is cheap insurance.

### F7. User describes a clearly unsafe or out-of-policy automation
- **Trigger**: Request involves monitoring a specific person without consent, scraping behind a paywall, mass-messaging external users, bypassing rate limits, etc.
- **Response**: Stop the skill. In Korean, explain the specific concern (e.g., "이 자동화는 [구체 사유]로 진행이 어려워 보여요") and suggest scoped alternatives. Do NOT write a PRD for the unsafe version. Offer to restart with a modified scope.
- **Why**: The skill should not launder unsafe requests into structured specs. Refusing at intake is far cheaper than refusing during implementation.

### F14. Quick-input mode (1~2 sentence entry) produces too-vague draft
- **Trigger**: User selected "한두 문장만 말할게요" in Step 1, but their 1-2 sentence input is too generic for Claude to confidently fill any 6-section guesses (e.g., "업무 자동화하고 싶어요" or "더 편하게 일하고 싶어요").
- **Response**: Do NOT proceed with a fully-fabricated draft. Reply once in Korean: "조금 더 구체적으로 알려주시면 더 좋은 초안을 만들어드릴 수 있어요. 예를 들어 (a) 어떤 업무가 (b) 얼마나 자주 (c) 어떤 결과물이 나오면 좋겠는지 한두 문장 더 부탁드려요." Wait for one more reply. If still too vague after second attempt, fall back to template mode (show the 6-section template) instead of guessing.
- **Why**: Quick-input mode is for users who have a *specific* idea but are reluctant to write a long brief. It is NOT for users who have no idea what they want — those need the template to anchor.

### F15. Build method "잘 모르겠어요" — Claude recommends but user wants both
- **Trigger**: In Step 3, user picks "잘 모르겠어요 — 추천해주세요" for build method. Claude suggests one (e.g., n8n based on PRD content). User replies "둘 다 만들고 싶어요" or "Claude Code도 해보고 싶고 n8n도 해보고 싶어요".
- **Response**: Pick ONE for this PRD. Explain in one Korean sentence: "이번 셋업에서는 한 가지만 정해서 끝까지 가야 6시간 안에 결과물이 나와요. 추천한 [방식]으로 우선 진행하고, 끝나면 다른 방식도 직접 시도해보실 수 있어요." If the user insists, default to the Claude recommendation and proceed.
- **Why**: Producing both `docs/PRD.md → code` and `docs/n8n-workflow.md` in parallel doubles cognitive load. 6시간 안에 한 가지를 *완성*하는 것이 두 가지를 *시작*하는 것보다 가치 높음.

## Mechanical failures

### F8. `Read` of `docs/PRD.md` or `CLAUDE.md` returns file-not-found in Step 0
- **Trigger**: Either Read returns an error indicating the path doesn't exist.
- **Response**: This is the happy path — proceed silently to Step 1 without asking the user about overwrites. Do NOT treat "not found" as an error worth surfacing.
- **Why**: First-time runs are the default case. Surfacing the absence as a warning makes the skill feel buggy.

### F9. `Bash mkdir -p docs` fails
- **Trigger**: Permission denied or filesystem error.
- **Response**: Stop the skill. Report the exact error in Korean: "docs 폴더를 만들지 못했어요. 다음 명령을 직접 실행해주시고 다시 시도해주세요: `mkdir -p docs`". Do NOT attempt creative workarounds (e.g., saving to a different path) — the save location is in the contract.
- **Why**: Silent path drift is the worst possible failure mode for a non-developer who later can't find their PRD.

### F10. User triggers the skill but their session is on a Claude Code version < v2.1.141
- **Trigger**: This skill cannot detect the version directly. If AskUserQuestion behaves visibly broken (popup covers previous chat), the user may complain.
- **Response**: If the user reports the popup is hiding earlier text, mention once: "Claude Code v2.1.141 이상으로 업데이트하면 이 화면이 더 잘 보여요. 일단 이대로 계속 진행할게요." Continue with the skill — version check is the user's problem to resolve later.
- **Why**: Skill must degrade gracefully on older versions. Recommend the upgrade, don't block on it.

## Recovery commands the user may issue

### F11. "처음부터 다시" / "처음부터 시작"
- **Response**: Save no files. End the current skill run with: "알겠습니다. `/hackathon-setup` 을 다시 실행해주세요." Do NOT auto-restart — let the user re-trigger.

### F12. "이전 단계로" / "Step N로 돌아가요"
- **Response**: This skill does not support arbitrary step jumps. Reply: "현재 흐름은 위에서 아래로만 진행돼요. 처음부터 다시 하시려면 `/hackathon-setup`을 다시 입력해주세요. 한두 항목만 고치고 싶으면 마지막 저장 후 직접 파일을 편집하셔도 돼요." This is a deliberate constraint, not a missing feature.

### F13. "취소" / "그만"
- **Response**: Exit immediately. If any files have been saved before the cancel, list them so the user knows what's on disk. If files have been backed up (`.bak`), mention that too. End with one line in Korean confirming the skill stopped.
