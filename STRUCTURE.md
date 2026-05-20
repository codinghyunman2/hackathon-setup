# `/hackathon-setup` 스킬 구조

> Anthropic Agent Skills의 Progressive Disclosure 패턴을 따르는 5-step 가이드형 스킬. 비개발자가 자동화 아이디어를 PRD 두 파일(docs/PRD.md, CLAUDE.md)로 구체화하고 `/plan`에 넘기는 것이 목적.

## 1. 디렉토리 트리

```
.claude/skills/hackathon-setup/
├── SKILL.md                              368 lines · 메인 워크플로우 (Steps 0~5)
├── README.md                             공개 README
├── LICENSE
├── .git/                                 독립 git repo (origin: codinghyunman2/hackathon-setup)
├── .gitignore
└── references/
    ├── failure_modes.md                  80 lines · F1~F15 gotchas 카탈로그
    ├── input_template_ko.md              86 lines · 사용자에게 보여줄 7섹션 한국어 템플릿
    ├── clarification_questions.md       170 lines · Step 2 보충 질문 풀 (Group A~F)
    ├── prd_template_en.md               150 lines · 영문 PRD 구조 (14 sections)
    ├── claude_md_template_en.md          51 lines · 영문 CLAUDE.md 구조 (slim, 3 sections)
    ├── n8n_workflow_template_en.md       84 lines · 영문 n8n workflow 구조
    └── examples/
        ├── example_prd.md               182 lines · PRD 모범 답안 (Weekly Marketing Bot)
        ├── example_claude_md.md          44 lines · CLAUDE.md 모범 답안
        └── example_n8n_workflow.md      201 lines · n8n workflow 모범 답안 (Store Review)
```

총 11개 파일.

## 2. 파일별 역할 & 로드 시점 (Progressive Disclosure 3-level)

| Level | When loaded | 파일 | 토큰 | 역할 |
|---|---|---|---|---|
| **1. Metadata** | 세션 시작 (always) | SKILL.md frontmatter | ~100 | name + description + disable-model-invocation |
| **2. Instructions** | `/hackathon-setup` 호출 시 | SKILL.md 본문 | ~2.7k | 5-step 워크플로우 control flow |
| **3. References** | Step 진입 시 on-demand | failure_modes.md | ~2.2k | Step 0 mandatory pre-read |
| | Template 모드만 | input_template_ko.md | ~1.0k | Step 1b-template에서 사용자에게 표시 |
| | Step 2 always | clarification_questions.md | ~1.5k | 보충 질문 풀 |
| | Step 3 always | prd_template_en.md + example_prd.md | ~3.5k | PRD 영문 번역 + quality bar |
| | Step 4 always | claude_md_template_en.md + example_claude_md.md | ~1.0k | CLAUDE.md slim 템플릿 + 예시 |
| | Step 4d only (n8n 선택 시) | n8n_workflow_template_en.md + example_n8n_workflow.md | ~3.6k | n8n 워크플로우 문서 |

**Full-run 토큰 합계**:
- Claude Code 경로: ~11.9k
- n8n 경로: ~15.5k

## 3. 워크플로우 흐름

```
[사용자가 /hackathon-setup 입력]
        ↓
┌────────────────────────────────────────────┐
│ Pre-read: failure_modes.md (mandatory)     │
└────────────────────────────────────────────┘
        ↓
┌─ Step 0 (Pre-flight) ──────────────────────┐
│ docs/PRD.md, CLAUDE.md 존재 체크           │
│  · 없음 → 무음 통과                        │
│  · 있음 → 덮어쓰기 Y/N + .bak 백업         │
└────────────────────────────────────────────┘
        ↓
┌─ Step 1 (입력) ────────────────────────────┐
│ 🎯 [1/5] 아이디어 입력 (1~5분)             │
│ ├─ 1a. 모드 선택 (퀵/템플릿/예시)         │
│ ├─ 1b-quick: 1~2문장 → Claude 7섹션 추측  │
│ └─ 1b-template: 7섹션 직접 입력           │
│    └ refers: input_template_ko.md          │
└────────────────────────────────────────────┘
        ↓
┌─ Step 2 (보충 질문) ───────────────────────┐
│ 🎯 [2/5] 빠진 정보 확인 (2~4분)            │
│ Round 1 (필수): A4 As-is 단계 + F1 권한    │
│ Round 1 슬롯 3 + Round 2 (최대): up to 3   │
│    └ refers: clarification_questions.md    │
└────────────────────────────────────────────┘
        ↓
┌─ Step 3 (PRD 작성) ────────────────────────┐
│ 🎯 [3/5] 자동화 설계서 만들기 (2~3분)      │
│ ├─ 3a. Silent draft (14 sections)         │
│ │  · build method 자동 추천 분기          │
│ │  · Section 0 자동 합성                  │
│ │  · [가정] 카운트 ≥5 → confirmation에 포함│
│ ├─ 3c. ★ ONE unified confirmation         │
│ │  (저장 / 수정 / 취소)                   │
│ │  · confirmation_round_count cap 3/5     │
│ └─ 3d. Save → docs/PRD.md (영문 번역)     │
│    └ refers: prd_template_en.md            │
│              examples/example_prd.md       │
└────────────────────────────────────────────┘
        ↓
┌─ Step 4 (CLAUDE.md + 조건부 n8n) ──────────┐
│ 🎯 [4/5] 클로드 규칙 파일 만들기 (2~4분)   │
│ ├─ 4a. API 문서 링크 수집                 │
│ ├─ 4b. Korean preview (slim 3 sections)   │
│ │  · 50줄 hard cap                        │
│ ├─ 4c. Confirm → CLAUDE.md (영문)         │
│ │    └ refers: claude_md_template_en.md   │
│ │              examples/example_claude_md │
│ └─ 4d. (build=n8n) → docs/n8n-workflow.md │
│      ├ Quality gate: 노드 ≥8/credential/  │
│      │  node type 정확성                  │
│      └ refers: n8n_workflow_template_en   │
│                examples/example_n8n_wf    │
└────────────────────────────────────────────┘
        ↓
┌─ Step 5 (Handoff) ─────────────────────────┐
│ 🎯 [5/5] 완료 — /plan 으로 넘기기          │
│ Build method에 따라 closing 분기:          │
│  · 분기 A (Claude Code): 2 파일 + /plan   │
│  · 분기 B (n8n): 3 파일 + n8n 빌드 가이드 │
│ ★ /plan 자동 트리거 절대 금지              │
└────────────────────────────────────────────┘
```

## 4. SKILL.md 본문 구조 (15 섹션)

| 라인 | 섹션 | 역할 |
|---|---|---|
| 1–5 | YAML frontmatter | name / description / disable-model-invocation |
| 7 | Surface note | Claude Code 전용 명시 |
| 9–11 | H1 + tagline | "Convert ... into PRD and CLAUDE.md" |
| 13–16 | Audience | 비개발자 정의 |
| 18–24 | Vocabulary mapping | docs/PRD.md → "자동화 설계서" 등 라벨 |
| 26–46 | Workflow 헤더 + Progress banner 정의 | 🎯 [N/5] 형식 명세 |
| 48–62 | Step 0 (Pre-flight) | 기존 파일 체크 |
| 64–95 | Step 1 (입력) | 1a entry mode + 1b-quick/template |
| 97–109 | Step 2 (보충 질문) | Round 1 필수 + 슬롯 + Round 2 |
| 111–175 | Step 3 (PRD 작성) | 3a silent draft / 3c confirmation / 3d save |
| 177–246 | Step 4 (CLAUDE.md + n8n) | 4a links / 4b preview / 4c save / 4d n8n |
| 248–308 | Step 5 (Handoff) | 분기 A/B closing + critical rule |
| 310–320 | References | 각 reference 파일 1줄 설명 |
| 322–334 | Recovery Commands | "처음부터 다시" 등 5종 핸들러 매핑 |
| 336–355 | Token Budget | 단계별 토큰 표 |
| 357–368 | Settings | 저장 경로, 라운드 캡, 언어 정책 등 |

## 5. 주요 설계 원칙 (Anthropic SSOT 정합)

| 원칙 | 본 스킬 구현 |
|---|---|
| **Progressive disclosure** | 메타 100t → SKILL 2.7k → 단계별 references on-demand |
| **References one level deep** | SKILL.md → references/*.md만, nested 없음 |
| **Failure capture loop** | F1~F15 gotchas 카탈로그 |
| **Template + Example pair** | 모든 영문 템플릿에 대응 example 파일 존재 |
| **Workflow checklist** | Step 0~5 + 🎯 banner |
| **Low/high freedom 분리** | Step 3c (low) / Step 1b-quick (high) |
| **disable-model-invocation** | 슬래시 전용 — 자동 호출 차단 |
| **Korean conversation / English files** | 대화는 한국어, 저장 파일은 영어 |
| **Slim CLAUDE.md** | ≤50줄 hard cap, 3 sections only |

## 6. 외부 의존성 & 인터페이스

| 항목 | 값 |
|---|---|
| 작동 환경 | Claude Code (CLI / 데스크탑 / VSCode Extension) |
| 최소 버전 | v2.1.141 (AskUserQuestion 팝업 버그) — `references/failure_modes.md` F10 |
| 입력 인터페이스 | `/hackathon-setup` 슬래시 명령어만 |
| 산출 파일 | `docs/PRD.md`, `CLAUDE.md`, (조건부) `docs/n8n-workflow.md` |
| 백업 | 기존 파일은 `.bak` 자동 백업 |
| 다음 단계 | `/plan` (사용자가 수동 입력 — 자동 트리거 금지) |
| 원격 저장소 | https://github.com/codinghyunman2/hackathon-setup |

---

이 구조의 핵심은 **"비개발자 한 명의 1회 대화로 다운스트림(/plan, 그 외 Claude Code 에이전트들)이 군더더기 없이 받을 수 있는 SSOT 두 파일을 만든다"** 입니다.
