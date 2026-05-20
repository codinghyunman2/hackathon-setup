# Example: Completed `docs/PRD.md` — Weekly Marketing Summary Bot

> Read this file in Step 3 BEFORE drafting the user's PRD. Treat it as the target quality bar — the user's final PRD should look this concrete, this specific, this scoped. Do not copy domain content; copy the level of detail.

This example was produced for a non-developer marketing manager who asked for "매주 월요일 광고 성과 슬랙으로 자동 전송".

## Contents

Example PRD body (inside the markdown code block below) — 14 sections:
- 0. Quick Facts & Hand-off (n8n / partial access risk / marketing-report)
- 1. Problem (5-step As-is workflow)
- 2. Target User (Performance Marketing Manager, non-developer)
- 3. Goals & Success Metrics
- 4. Scope (In scope / Out of scope)
- 5. Inputs & Data Sources (Meta Ads / Google Ads / Airbridge / Slack)
- 6. Output (Slack Block Kit message structure)
- 7. Deployment & Usage
- 8. Constraints & Non-Negotiables
- 9. References
- 10. Hackathon MVP (Meta Ads only, single trigger)
- 11. Phase 2 (Google Ads + Airbridge, slash command, Block Kit button)
- 12. Access Risk & Pre-requisites
- 13. Build Method (n8n, with rationale)
- 14. Assumptions

After the code block:
- "What makes this example 100점 PRD" — 10 quality criteria

---

```markdown
# PRD — Weekly Marketing Summary Bot

> Every Monday 9 AM, automatically post last week's ad campaign performance to the #marketing-weekly Slack channel so the marketing manager doesn't have to spend 1.5 hours compiling it by hand.

## 0. Quick Facts & Hand-off (frontmatter for reviewers + downstream agents)

> Single source of truth that `/plan` and every downstream Claude Code session reads first.

**Quick Facts**
- **Build method**: n8n (self-hosted, marketing team already operates an n8n instance)
- **Hackathon MVP**: Trigger once a week, pull Meta Ads weekly totals only, post a static Slack message to #marketing-weekly. Google Ads + Airbridge deferred to Phase 2.
- **Access risk**: partial — Meta Ads token confirmed available; Google Ads / Airbridge tokens need request before any integration work
- **Category tag**: marketing-report

**Hand-off rules for `/plan` and downstream agents**
- **User technical level**: non-developer — default to n8n nodes and Google Sheet templates. Avoid Docker, microservices, framework boilerplate.
- **Language policy**: explain in Korean. Code, file names, commit messages, n8n node names stay in English.
- **Credential-verification rule**: Access risk is `partial`. First `/plan` step MUST be a Meta Ads token smoke test. Do NOT scaffold the full pipeline before that smoke test passes.
- **Done = Success Criteria in section 3** (bot runs every Monday with zero manual edits for 4 consecutive weeks).
- **Plan sequencing**: test after each n8n node addition — never bundle a 5-node import with all credentials in one step.

## 1. Problem

- Every Monday morning, the marketing manager manually copies last week's ad performance from 3 dashboards (Meta Ads, Google Ads, Airbridge) into a Slack message for the team lead.
- It takes 1–1.5 hours and happens 52 times a year (~70 hours/year of manual work).
- Currently solved by: opening 3 dashboard tabs, exporting CSVs, pasting numbers into a Slack draft. Frequent copy-paste errors. Team lead has flagged 2 incidents in the last quarter where the wrong week's data was posted.
- **As-is workflow**:
  1. Open Meta Ads Manager → set date range to last week → export CSV
  2. Open Google Ads → set date range → export CSV
  3. Open Airbridge dashboard → filter by campaign → screenshot
  4. Paste numbers into a Google Sheet template → recalculate ROAS/CPI
  5. Copy the formatted text into Slack `#marketing-weekly` and post

## 2. Target User

- Primary user: Performance Marketing Manager (1 person, marketing team of 4)
- Technical skill level: **non-developer** — comfortable with Google Sheets, Slack, no coding background
- Where in workflow: Monday 9–10 AM, before the weekly marketing standup at 11 AM

## 3. Goals & Success Metrics

- Primary goal: Reduce Monday morning manual reporting from 1.5 hours to under 5 minutes (review only).
- Success metric: 4 consecutive weeks with zero manual edits before the message is posted.
- Hackathon "done" definition: Bot runs automatically every Monday 9 AM, posts a correctly formatted summary to #marketing-weekly, includes a "데이터 확인 완료" button the manager clicks to confirm.

## 4. Scope

### In scope (MVP)
- Pull last 7 days of metrics from Meta Ads, Google Ads, Airbridge via their CSV export or API
- Aggregate into: total spend, total installs, CPI, ROAS, top-3 best-performing campaigns
- Post a single Slack message with the summary, formatted as a Block Kit message
- Manual override: manager can trigger an off-schedule run via a Slack slash command `/marketing-summary now`

### Out of scope (explicitly excluded)
- Real-time alerts (no streaming, no anomaly detection)
- Historical comparison beyond last-week-vs-week-before
- Channels other than #marketing-weekly
- Mobile push notifications
- Dashboard UI — Slack message is the only surface

## 5. Inputs & Data Sources

- Meta Ads Manager: campaign-level daily spend, impressions, clicks, installs (via Meta Marketing API)
- Google Ads: campaign-level daily spend, impressions, clicks (via Google Ads API)
- Airbridge: install events, revenue events grouped by campaign (via Airbridge Raw Data Export API)
- Run cadence: every Monday 9:00 AM KST, plus on-demand via slash command
- Connected tools: Meta Marketing API, Google Ads API, Airbridge API, Slack Web API

## 6. Output

- Output artifact: a single Slack message in #marketing-weekly using Block Kit
- Structure:
  - Header: "📊 지난주 마케팅 성과 (YYYY-MM-DD ~ YYYY-MM-DD)"
  - Section 1: Total spend, installs, CPI, ROAS (4 number tiles)
  - Section 2: Top 3 campaigns by ROAS (campaign name, spend, ROAS)
  - Footer: "데이터 확인 완료" button → marks the run as reviewed
- Delivery destination: Slack channel `#marketing-weekly` (channel ID: `C0XXXXXX`)

## 7. Deployment & Usage

- Team-internal (marketing team of 4, plus 1 team lead)
- Deployment target: single Python script running on a cron schedule via internal scheduler (preferred) OR n8n self-hosted workflow if no-code is more maintainable for the user
- The marketing manager will own day-to-day operation; engineering will own initial deployment only

## 8. Constraints & Non-Negotiables

- Must NOT post if any of the 3 data sources fails — instead, post a "데이터 수집 실패" alert message to the manager's DM with the specific source that failed
- Must NOT auto-delete or edit previous Monday messages (audit trail)
- API credentials must live in environment variables, never in code or Slack

## 9. References

- Existing manual template the team uses (currently in Notion page `marketing-weekly-template`)
- Slack Block Kit reference: https://api.slack.com/block-kit
- [Assumption: existing Notion template URL not provided; will request during Step 4 setup]

## 10. Hackathon MVP (6-hour scope)

- ONE trigger: cron job at Monday 9:00 AM KST.
- ONE data source for MVP: Meta Ads weekly totals (spend, impressions, clicks). Google Ads + Airbridge are explicitly Phase 2.
- ONE output: a static plain-text Slack message to `#marketing-weekly` containing the 4 number tiles. No Block Kit button, no slash command — those move to Phase 2.
- Acceptance: at the end of the 6-hour window, the bot has successfully run end-to-end at least once against last week's real Meta Ads data and posted to the channel.

## 11. Phase 2 (Post-Hackathon)

- Google Ads + Airbridge integration (require credential request, ~2-3 days lead time)
- Block Kit message with "데이터 확인 완료" button
- Slash command `/marketing-summary now` for on-demand runs
- Top-3 best-performing campaigns section
- Failure DM alert when any data source is unavailable
- Last-week-vs-week-before comparison

## 12. Access Risk & Pre-requisites

- Meta Ads Marketing API token — confirmed available (manager already has admin access)
- Google Ads API token — **needs request** (must file IT ticket; ~2-day SLA). Hackathon MVP avoids this dependency.
- Airbridge Raw Data Export API — **needs verification** with Airbridge admin whether the plan includes this feature. Hackathon MVP avoids this dependency.
- Slack bot token + `#marketing-weekly` write permission — needs creation but doable in 30 min during hackathon
- **First /plan step**: confirm Meta Ads token works in a smoke test before any further build steps.

## 13. Build Method

- **Selected**: n8n
- **Why this fits**: Marketing team already operates a self-hosted n8n instance, and the workflow is dominated by external API integrations (Meta → Slack) with light transformation — n8n's HTTP Request + Slack nodes cover this without custom code. Non-developer manager can maintain the workflow visually after handoff.
- **If n8n is selected**: a companion file `docs/n8n-workflow.md` will be produced alongside this PRD.

## 14. Assumptions

- [Assumption: Marketing team uses Slack as primary channel — confirmed in Step 2]
- [Assumption: Airbridge Raw Data Export API is enabled on the team's plan — needs verification with Airbridge admin (Phase 2)]
- [Assumption: Meta Ads API credentials are obtainable by the user — confirmed for MVP]

> Generated by `/hackathon-setup`. Run `/plan` next to turn this PRD into an execution plan.
```

---

## What makes this example "100점 PRD"

1. **Section 0 (Quick Facts) is filled in 4 lines** — reviewers can scan 300 PRDs by section 0 alone.
2. **Concrete numbers everywhere** — "1.5 hours", "52 times a year", "4 consecutive weeks", not vague "frequently" or "saves time".
3. **Named channel / API / column** — `#marketing-weekly`, `Airbridge Raw Data Export API`, `데이터 확인 완료 button` — `/plan` can act on these directly.
4. **As-is workflow has 5 explicit steps** — the automation target is now visible (steps 1–3 = data collection automation point).
5. **Out of scope section is longer than 2 bullets** — non-developers tend to leak scope; show them explicitly what NOT to build.
6. **Section 10 MVP is single-trigger / single-source / single-output** — fits 6 hours. Everything else moved to section 11 Phase 2 instead of being silently dropped.
7. **Section 12 Access Risk lists each dependency with status** — `/plan` first step is auto-determined.
8. **Section 13 Build Method has a one-line justification** — not just "n8n" but "n8n because 4+ API integrations".
9. **Constraints section has must-NOT rules** — failure modes are pre-decided, not deferred to `/plan`.
10. **Assumptions are sharp** — each one is a single verifiable claim, not a vague hedge.

When the user's actual answers are vague ("매주 빠르게 보내고 싶어요"), do NOT fabricate this level of detail. Use `[Assumption: ...]` markers liberally and let `/plan` resolve them. But aim for this section structure and this tone.
