# hackathon-setup

> 비개발자(마케터, PM, 운영, 디자이너)가 자동화 아이디어를 **PRD + CLAUDE.md**로 5분 만에 정리해 `/plan`으로 넘길 수 있게 도와주는 Claude Code 스킬

`/hackathon-setup` 한 번이면 한국어 대화로 6섹션 템플릿을 채우고, 최대 6개 후속 질문을 받은 뒤, 다운스트림 `/plan` 이 바로 소비할 수 있는 영문 스펙 2개(`docs/PRD.md`, 프로젝트 루트 `CLAUDE.md`)를 자동 생성합니다.

---

## Who is this for

- 사내 Claude Code mini-hackathon 참가자
- 코딩 경험이 없거나 적은 직군 — 마케터, PM, 운영, 디자이너
- "AI에게 뭘 어떻게 시켜야 할지 모르겠다"는 빈 프롬프트 공포가 있는 모든 사람

## What it does

| Step | 무엇을 | 예상 시간 |
|---|---|---|
| 0 | 기존 파일 충돌 확인 (덮어쓸지 Y/N) | 1분 이내 |
| 1 | 6섹션 한국어 템플릿 입력 (자유 텍스트) | 3~5분 |
| 2 | 최대 2라운드 × 3개 후속 질문 (AskUserQuestion) | 2~4분 |
| 3 | `docs/PRD.md` 한국어 미리보기 → 영문 저장 | 2~3분 |
| 4 | `CLAUDE.md` 한국어 미리보기 → 영문 저장 (API 문서 링크도 함께) | 2~3분 |
| 5 | `/plan` 으로 핸드오프 (수동 트리거 — 자동 실행 안 함) | — |

**핵심 특징**
- 화면 대화는 **한국어**, 저장 파일은 **영어** (Claude가 영어 스펙을 더 정확히 따라가서)
- 모든 사용자 질문은 `AskUserQuestion` 도구로만 — plain-text A/B/C 형식 금지 (자유 입력은 각 질문의 "Other"로 가능)
- 진행 표시기 `🎯 [N/5]`로 비개발자가 위치를 항상 알 수 있음
- 품질 검증 게이트: 필수 섹션 누락·`[가정: ...]` 마커 5개 이상 시 사용자에게 한 번 더 확인
- `/plan` 자동 트리거 금지 — 비개발자가 CLAUDE.md 검토 후 직접 입력
- 13개 엣지 케이스 카탈로그(`references/failure_modes.md`)로 대화 이탈·언어 전환·부적절 요청 등 처리

## Installation

이 저장소를 본인의 Claude Code 프로젝트의 `.claude/skills/hackathon-setup/` 경로로 클론하세요.

```bash
# 프로젝트 루트에서
mkdir -p .claude/skills
git clone https://github.com/codinghyunman2/hackathon-setup.git .claude/skills/hackathon-setup
```

또는 글로벌 사용자 스킬로:

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/codinghyunman2/hackathon-setup.git ~/.claude/skills/hackathon-setup
```

**요구사항**
- Claude Code `v2.1.141` 이상 (이전 버전은 AskUserQuestion 팝업이 이전 대화를 가리는 버그가 있음)

## Usage

이 스킬은 `disable-model-invocation: true` 로 설정되어 있어 **사용자가 직접 슬래시 커맨드를 입력해야만** 동작합니다. Claude가 자연어 대화에서 자동으로 호출하지 않습니다.

```text
/hackathon-setup
```

**이 커맨드를 입력하면 좋은 상황** (입력은 항상 위 한 줄로):
- 사내 해커톤에서 자동화 아이디어를 정리하고 싶을 때
- "자동화 PRD 만들어줘" 같은 작업이 필요하다고 느낄 때
- Claude에게 뭘 어떻게 시켜야 할지 막막할 때

스킬이 끝나면 다음 출력이 표시됩니다.

```text
셋업이 끝났어요! 🎉

생성된 파일:
- 자동화 설계서: docs/PRD.md
- 클로드 규칙 파일: CLAUDE.md

👉 다음으로 할 일 (순서 중요!)
1. CLAUDE.md 를 열어서 내용을 한 번 훑어봐 주세요.
2. 내용이 마음에 들면, 직접 /plan 을 입력해 주세요.
```

## Repository structure

```
hackathon-setup/
├── SKILL.md                              # 메인 워크플로 정의 (Claude Code 진입점)
├── README.md                             # 이 파일
└── references/
    ├── input_template_ko.md              # Step 1 — 6섹션 한국어 입력 템플릿
    ├── clarification_questions.md        # Step 2 — 후속 질문 풀
    ├── prd_template_en.md                # Step 3 — 영문 PRD 구조
    ├── claude_md_template_en.md          # Step 4 — 영문 CLAUDE.md 구조
    ├── failure_modes.md                  # F1~F13 엣지 케이스 카탈로그
    └── examples/
        ├── example_prd.md                # 100점 PRD 모범 사례 (마케팅 보고 자동화)
        └── example_claude_md.md          # 짝지어진 CLAUDE.md 모범 사례
```

## What this skill is NOT

- `/plan` 을 대체하지 않습니다. 이 스킬은 PRD를 만들고 멈춥니다.
- 코드를 직접 작성하지 않습니다. 다음 단계는 사용자가 직접 `/plan` 을 호출해야 시작됩니다.
- 다국어 옵션이 없습니다. 화면은 항상 한국어, 파일은 항상 영어입니다.
- 한 번 실행에 한 프로젝트만 다룹니다. Step 2 도중 다른 아이디어로 분기하면 거절하고 원래 프로젝트로 돌아옵니다.

## Recovery commands

사용자가 다음과 같이 말하면 스킬이 대응합니다(자세한 핸들러는 `references/failure_modes.md` 참고):

| 사용자 발화 | 동작 |
|---|---|
| "처음부터 다시", "리셋" | 종료 후 재호출 안내 |
| "이전 단계로", "Step 2로 돌아가요" | 선형 흐름임을 안내, 재호출 또는 직접 편집 권유 |
| "취소", "그만" | 즉시 종료, 저장된 파일 위치 안내 |
| "/plan 자동으로 돌려줘" | 거절 (Step 5 핵심 규칙) |

## Why it scored ~95/100

이 스킬은 다음 상위 1% Claude Code 스킬 패턴을 따릅니다.

1. **Progressive disclosure** — `SKILL.md`는 짧고, 상세 내용은 `references/` 로 분리
2. **Worked examples** — `references/examples/` 에 완성된 PRD + CLAUDE.md 쌍이 LLM 모방 대상으로 존재
3. **Quality validation gates** — Step 3·4 저장 전 필수 섹션·가정 마커 카운트·핸드오프 섹션 강제 검증
4. **Inter-skill handoff** — CLAUDE.md `## Hand-off Notes` 섹션에 `/plan` 과 모든 다운스트림 세션이 자동 읽을 메타 지시 포함
5. **Failure mode catalog** — 13개 엣지 케이스 명시 처리 (F1~F13)
6. **Token budget transparency** — 풀런 약 10k 토큰 적재량 공시
7. **Idempotent re-runs** — Step 0에서 기존 파일을 자동 `.bak` 백업
8. **Explicit recovery commands** — 5개 사용자 발화에 대한 명시적 라우팅 테이블

## Contributing

이 스킬은 사내 Claude Code mini-hackathon용으로 만들어졌지만, 비개발자 대상 자동화 셋업 시나리오라면 누구나 활용할 수 있도록 공개합니다.

개선 제안은 이슈로 남기거나 PR 보내주세요. 특히 환영합니다:
- 더 다양한 도메인의 worked example (현재는 마케팅 1개)
- 새로운 failure mode 케이스 (F14 이상)
- 다른 언어 대응 가이드 (현재 한국어 1개)

## License

MIT
