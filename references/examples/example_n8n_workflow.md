# Example: Completed `docs/n8n-workflow.md` — Store Review Auto-Response

> Read this in Step 4d BEFORE drafting the user's n8n workflow document (only when build method = n8n). Mimic the specificity (real n8n node types from the actual workflow JSON, real field configs, real credentials), not the domain content.

This example is based on a real production n8n workflow ("Store Review") used at a mobile game studio to auto-reply to Google Play / App Store reviews across multiple languages using a Gemini LLM chain. It is **standalone** — the workflow doc here is self-contained and does not depend on `example_prd.md`.

## Contents

Outer sections (this file):
- Mini-PRD Context (problem / goal / MVP / build method rationale)
- Example workflow body (inside the markdown code block below)
- What makes this example "100점 n8n workflow doc" — quality criteria

Workflow body sections (inside the code block):
- Trigger (Schedule Trigger configuration)
- Node Sequence (happy path — full node-by-node walkthrough with Node type / Purpose / Key config / Output)
- Branches & Error Handling (MVP minimum — empty data + failure cases)
- Required Credentials (n8n Credentials store table)
- Estimated Complexity (node count / API count / build time / difficulty)
- What this workflow does NOT cover (Phase 2 carryover from PRD section 11)
- Visual Reference (ASCII node chain)

---

## Mini-PRD Context (for this example only)

- **Problem**: CS/PM team manually replies to 50+ user reviews per day across 2 games (꿈틀 수비대 / 벙커 디펜스) and multiple languages. ~2 hours/day of copy-paste + translation work, frequent tone inconsistency.
- **Goal**: Auto-detect each review's language, generate a tone-controlled 4-sentence reply in the SAME language, categorize for the dev team, post the reply back, log to a tracking sheet.
- **Hackathon MVP**: Replies via internal aggregator API only (not direct Play/Store APIs). One internal POST endpoint already exists. Schedule 3× per day (9 AM / 3 PM / 9 PM KST). Gemini via Google Vertex.
- **Build method**: n8n (selected because 4 external integrations + LLM call dominate the workflow — exactly n8n's sweet spot).

---

```markdown
# n8n Workflow — Store Review Auto-Response

> Companion to `docs/PRD.md`. Implements the Hackathon MVP: 3× daily fetch of unanswered store reviews, LLM-generated multilingual reply, POST back, log to Google Sheet. Phase 2 capabilities (direct Google Play API, App Store Connect API, Slack escalation for [기술적 결함] priority) are NOT included.

## Trigger

- **Trigger type**: Schedule Trigger
- **Schedule / event**: 3 times daily — 09:00, 15:00, 21:00 KST (every day of the week)
- **Why this trigger**: PRD specifies 3× daily cadence to balance reply latency vs. API rate limits and CS team review burden.

## Node Sequence (happy path)

1. **Schedule Trigger** — `n8n-nodes-base.scheduleTrigger`
   - Purpose: fire the workflow 3× per day at fixed hours.
   - Key config:
     - Trigger Rule: Weeks → Days `[Mon, Tue, Wed, Thu, Fri, Sat, Sun]` → Hours `[9, 15, 21]`
     - Timezone: `Asia/Seoul`
   - Output: empty payload (just a timestamp).

2. **GET-Review** — `n8n-nodes-base.httpRequest`
   - Purpose: fetch the latest unanswered reviews from the studio's internal review aggregator API.
   - Key config:
     - Method: `GET`
     - URL: `https://auto-reply.spartagames.kr/api/v1/reviews/unanswered`
     - Query Parameters: `limit = 50`
     - Batching: batchSize `1`, batchInterval `500ms` (gentle on the upstream)
   - Output: `[ { reviewId, appName, store, packageName, totalContent, rating, ... }, ... ]` — an array of review objects.

3. **Split Out** — `n8n-nodes-base.splitOut`
   - Purpose: iterate one review at a time so subsequent LLM calls process them individually.
   - Key config:
     - Field to split out: `data.reviews`
   - Output: one item per review.

4. **Wait1** — `n8n-nodes-base.wait`
   - Purpose: 30-second pause between LLM calls to respect Gemini rate limits and let async POSTs settle.
   - Key config:
     - Amount: `30` (seconds)
   - Output: pass-through with delay.

5. **Basic LLM Chain** — `@n8n/n8n-nodes-langchain.chainLlm`
   - Purpose: generate the reply text + classification in one structured call.
   - Sub-node attached: **Google Vertex Chat Model** (`@n8n/n8n-nodes-langchain.lmChatGoogleVertex`) using `models/gemini-2.0-flash`, temperature `0.4`.
   - Sub-node attached: **Structured Output Parser** (`@n8n/n8n-nodes-langchain.outputParserStructured`) with JSON schema:
     ```json
     {
       "reply": "유저에게 보낼 답글 내용입니다.",
       "category": "기술적 결함",
       "priority": "High",
       "reason": "로그인 불가 현상을 언급하여 기술적 결함으로 분류함"
     }
     ```
   - Key config (system prompt — abbreviated; see workflow JSON for full text):
     - Role: "professional game developer for `{{ $json.appName }}` and a data analyst"
     - **CRITICAL RULES**:
       1. LANGUAGE MATCHING — detect review language, reply in that same language only
       2. STRICT LENGTH — max 200 characters including spaces, single line (no `\n`)
       3. STRUCTURE — exactly 4 sentences (Thanks → Contextual empathy → Situational guideline → Closing)
       4. NO EMOJIS; Korean uses `-습니다/합니다` style
     - **CATEGORIES**: `[구체적 페인포인트]` / `[업데이트 이슈]` / `[기술적 결함]` / `[경쟁작 비교]` / `[일반]`
     - **PRIORITY**: High (Technical/Update issue) | Medium | Low
     - Batching: batchSize `1`
     - Retry: `retryOnFail: true`, `waitBetweenTries: 5000`, `maxTries: 5`
   - Output: `{ output: { reply, category, priority, reason } }`.

6. **POST-Review** — `n8n-nodes-base.httpRequest`
   - Purpose: submit the generated reply back to the internal aggregator, which routes to Google Play / App Store.
   - Key config:
     - Method: `POST`
     - URL: `https://auto-reply.spartagames.kr/api/v1/reviews/reply`
     - Headers: `Content-Type: application/json`
     - Body params:
       - `replyText` = `{{ $json.output.reply }}`
       - `store` = `{{ $('Split Out').item.json.store }}`
       - `reviewId` = `{{ $('Split Out').item.json.reviewId }}`
       - `packageName` = `{{ $('Split Out').item.json.packageName }}`
     - Batching: batchSize `1`, batchInterval `2000ms`
   - Output: `{ success: true/false, data: { repliedAt, appName, store, ... } }`.

7. **Append row in sheet** — `n8n-nodes-base.googleSheets`
   - Purpose: log every reply attempt to the team's tracking sheet for CS review and metrics.
   - Key config:
     - Operation: `append`
     - Document: `Auto Store Review - Record` (sheet ID `1XhUTvyp_A0wZqiEjk8wI2Rb6-DmGWE8vrFCnLcBkOik`)
     - Sheet: `시트1` (gid=0)
     - Column mapping:
       - `Date` ← `{{ $json.data.repliedAt }}`
       - `Status` ← `{{ $json.success }}`
       - `App Name` ← `{{ $json.data.appName }}`
       - `Store` ← `{{ $json.data.store }}`
       - `Category` ← `{{ $('Basic LLM Chain').item.json.output.category }}`
       - `Priority` ← `{{ $('Basic LLM Chain').item.json.output.priority }}`
       - `rating` ← `{{ $('Split Out').item.json.rating }}`
       - `Content` ← `{{ $('Split Out').item.json.totalContent }}`
       - `reason` ← `{{ $('Basic LLM Chain').item.json.output.reason }}`
   - Output: append confirmation.

## Branches & Error Handling (MVP minimum)

- **LLM transient failures**: the Basic LLM Chain has `retryOnFail: true` with 5 attempts × 5s backoff. After 5 failures, the item is dropped (logged to n8n execution history). Phase 2 adds Slack DM alert on persistent failure.
- **POST failure**: if `POST-Review` returns non-200, `success` field is `false`. The Google Sheets log still records the attempt with `Status = false` so the CS team can manually intervene. No retry on POST in MVP — failed replies are visible in the sheet within minutes.
- **Empty input case**: if `GET-Review` returns 0 reviews, `Split Out` produces 0 items → downstream nodes do not execute. n8n execution log shows "no items" — silent success, no Slack noise.
- Rate-limit handling beyond the 30s `Wait1`, exponential backoff on POST, dead-letter queue → all in Phase 2.

## Required Credentials (n8n Credentials store)

| Credential name in n8n | Type | Where to obtain | Already available? |
|------------------------|------|-----------------|-------------------|
| `Google Service Account account - Gemini` | Google Service Account (Vertex AI) | GCP IAM → Service Accounts → JSON key with Vertex AI User role | yes (already in n8n instance) |
| `Google Sheets account` | OAuth2 (Google Sheets API) | n8n Credentials → Google Sheets → OAuth flow | yes |
| (Internal API `auto-reply.spartagames.kr`) | No credential — public internal endpoint | n/a | n/a |

**Pre-flight**: confirm the Vertex Service Account has access to the GCP project `gen-lang-client-0034471667` (the project ID embedded in the Vertex node config) before importing the workflow.

## Estimated Complexity

- **Node count**: 7 (Schedule Trigger → GET-Review → Split Out → Wait1 → Basic LLM Chain [+2 sub-nodes: Vertex Chat Model, Structured Output Parser] → POST-Review → Google Sheets append)
- **External APIs touched**: 3 (Internal aggregator GET + POST, Google Vertex Gemini, Google Sheets)
- **Build time estimate** (excluding credential wait): ~3 hours (most of the time is iterating on the LLM system prompt for tone/length compliance)
- **Difficulty for a non-developer**: Medium — drag-and-drop for 5 of the 7 nodes, but the LLM Chain prompt needs careful editing and the Structured Output Parser schema must match the prompt's JSON output format exactly. Pair-program with the CS/PM domain expert who knows the desired reply tone.

## What this workflow does NOT cover (see PRD Phase 2)

- Direct Google Play Android Publisher API integration (currently routed through internal aggregator)
- Direct App Store Connect API integration
- Slack escalation when `priority = High` and `category = 기술적 결함` — currently CS team scans the Google Sheet manually
- Per-app prompt customization (꿈틀 수비대 vs 벙커 디펜스 currently share one system prompt)
- A/B testing different reply templates
- Reply quality scoring (human-in-the-loop QA)

## Visual Reference

```
[Schedule Trigger (3×/day)]
        ↓
[GET-Review (internal aggregator API)]
        ↓
[Split Out — per review]
        ↓
[Wait1 — 30s]
        ↓
[Basic LLM Chain] ← [Google Vertex Chat Model]
        │           ← [Structured Output Parser]
        ↓
[POST-Review (internal aggregator API)]
        ↓
[Append row in sheet (Google Sheets)]
```

> Generated by `/hackathon-setup`. Edit this file when you tweak the n8n graph in the UI — keep them in sync.
```

---

## What makes this example "100점 n8n workflow doc"

1. **Every node uses the exact n8n type identifier** — `n8n-nodes-base.scheduleTrigger`, `@n8n/n8n-nodes-langchain.chainLlm`, etc. A non-developer can paste these into the n8n node search.
2. **LLM Chain shows sub-node composition** — Basic LLM Chain has 2 attached sub-nodes (Chat Model + Output Parser). Without this clarity, non-developers wire the parts in series and the chain fails.
3. **Structured Output Parser schema is shown verbatim** — JSON shape matches the system prompt's `[OUTPUT FORMAT]` exactly. Mismatched schema vs prompt = the #1 silent failure for LLM Chain users.
4. **System prompt is summarized in 4 critical rules + categories** — not pasted in full (would balloon to 100 lines), but specific enough that a CS PM can audit tone policy.
5. **Real URLs / IDs from production** — `auto-reply.spartagames.kr`, sheet ID `1XhUTvyp_...`, GCP project `gen-lang-client-0034471667`. Non-developers don't have to invent placeholder values.
6. **7 nodes — exactly MVP scope** — Filter / Limit / Wait / Code / direct Play API nodes from the production workflow are explicitly listed as Phase 2.
7. **Retry config is visible** — `retryOnFail: true`, `maxTries: 5`, `waitBetweenTries: 5000`. LLM workflows without retry fail constantly in production; calling this out in MVP prevents an obvious post-hackathon outage.
8. **Credentials table maps to GCP IAM roles** — `Vertex AI User` is the specific role needed. Pre-flight check is one IAM lookup, not a 30-minute discovery session.
9. **Visual reference shows the sub-node attachment with arrows** — Chat Model + Output Parser are visually connected sideways to the LLM Chain, mirroring the n8n UI shape.

When the user's actual answers are sparser, keep this 7-node ceiling and this credentials-table format. Hackathon MVP n8n workflows that exceed 8 nodes almost always include Phase 2 scope leakage.
