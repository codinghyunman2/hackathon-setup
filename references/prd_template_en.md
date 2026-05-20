# PRD Template (English) — `docs/PRD.md`

Use this exact structure when translating the confirmed Korean summary to English in Step 3. Keep section headings as written. Preserve every `[가정: ...]` marker by translating it to `[Assumption: ...]`.

## Contents

Template body (inside the markdown code block below) — 14 sections plus frontmatter:
- 0. Quick Facts & Hand-off (Build method / Hackathon MVP / Access risk / Category tag + Hand-off rules)
- 1. Problem (with As-is workflow)
- 2. Target User
- 3. Goals & Success Metrics
- 4. Scope (In scope / Out of scope)
- 5. Inputs & Data Sources
- 6. Output
- 7. Deployment & Usage
- 8. Constraints & Non-Negotiables
- 9. References
- 10. Hackathon MVP (6-hour scope)
- 11. Phase 2 (Post-Hackathon)
- 12. Access Risk & Pre-requisites
- 13. Build Method
- 14. Assumptions

After the code block:
- Translation guidance (preservation rules, mandatory sections, final line)

---

```markdown
# PRD — {Project name in English}

> One-line summary of what this automation does, written from the user's perspective.

## 0. Quick Facts & Hand-off (frontmatter for reviewers + downstream agents)

> This is the single source of truth that `/plan` and every downstream Claude Code session reads first. Keep each line tight.

**Quick Facts**
- **Build method**: {Claude Code | n8n | TBD — see section 13}
- **Hackathon MVP**: {one-line description of what will be done in the 6-hour window}
- **Access risk**: {none | partial — see section 12 | unknown — must verify first}
- **Category tag**: {e.g., marketing-report | hr-onboarding | cs-triage | sales-followup | ops-monitoring}

**Hand-off rules for `/plan` and downstream agents** (read these before suggesting any code)
- **User technical level**: {non-developer | developer} — if non-developer, default to no-code or single-file scripts; avoid microservices, Docker, framework boilerplate.
- **Language policy**: explain in Korean (conversation, plan steps, prose). Code, file names, function names, commit messages stay in English.
- **Credential-verification rule**: before writing any integration code, confirm the user actually has the credential. If Access risk above is `partial` or `unknown`, the first plan step must be a credential verification step.
- **Done = Success Criteria in section 3.** Do not invent additional definitions of done.
- **Plan sequencing**: prefer recoverable, testable-after-each-step changes over bundled validation at the end.

## 1. Problem

- What is the current pain point?
- How often does it happen and how much time/effort does it cost?
- How is it being solved today (manual? spreadsheet? other tool?)?
- **As-is workflow (3~5 step list)** — concrete sequence the user performs by hand today.

## 2. Target User

- Primary user (role, team, headcount)
- Technical skill level — explicitly note "non-developer" when applicable
- Where in their workflow this automation fits

## 3. Goals & Success Metrics

- Primary goal in one sentence
- Concrete success metric (time saved, error rate reduced, frequency of use, etc.)
- Definition of "done" for the hackathon scope (must match section 10 MVP)

## 4. Scope

### In scope (Hackathon MVP — must fit in 6 hours)
- Capability 1
- Capability 2
- ...

### Out of scope (explicitly excluded)
- Feature deferred to later
- Anything the user marked as "절대 안 됐으면 하는 것"

## 5. Inputs & Data Sources

- Where data comes from (platforms, files, URLs, APIs)
- Run cadence (daily / weekly / on-demand)
- Connected tools (Meta, Airbridge, Notion, Google Sheets, Slack, etc.)

## 6. Output

- Output artifact format (file type / message / dashboard)
- Delivery destination (Slack channel, email, Google Sheet, local file, web URL)
- Sample structure or example if available

## 7. Deployment & Usage

- Single user / team-internal / public
- Headcount
- Deployment target (local execution / internal server / external host like Vercel / n8n self-hosted)

## 8. Constraints & Non-Negotiables

- Hard constraints from the user (security, privacy, data residency, vendor lock-in)
- Things that must NOT happen

## 9. References

- Comparable products or examples the user mentioned

## 10. Hackathon MVP (6-hour scope)

- The single smallest end-to-end version that delivers value in 6 hours of work.
- Must be testable: one trigger, one happy-path output, no edge cases.
- This is what `/plan` will optimize for. Anything else lives in section 11.

## 11. Phase 2 (Post-Hackathon)

- Features the user wants but does NOT fit in 6 hours.
- Listed here intentionally so they are not lost, but `/plan` will not implement them in the hackathon scope.
- Examples: edge case handling, advanced filters, multi-channel delivery, retro analytics.

## 12. Access Risk & Pre-requisites

- What system access / credentials are required (Slack token, Google Sheets API, internal DB account, ...)
- Current status: {confirmed available | needs request | unknown}
- If status is anything other than "confirmed available", the first `/plan` step must be a verification/request step before any code is written.

## 13. Build Method

- **Selected**: {Claude Code | n8n | TBD}
- **Why this fits** (one line): e.g., "n8n suits this because 4+ external API integrations dominate the workflow" / "Claude Code suits this because custom data transformation logic is needed".
- **If n8n is selected**: a companion file `docs/n8n-workflow.md` will be produced alongside this PRD.

## 14. Assumptions

- [Assumption: ...]
- [Assumption: ...]

> Generated by `/hackathon-setup`. Run `/plan` next to turn this PRD into an execution plan.
```

---

## Translation guidance

- Preserve all `[가정: ...]` markers — convert to `[Assumption: ...]`.
- Keep proper nouns (Airbridge, Meta, Notion, etc.) as-is.
- Translate Korean role names to natural English (예: "성과 마케터" → "Performance Marketer").
- When a Korean bullet is ambiguous, prefer the conservative interpretation rather than over-specifying — `/plan` will surface uncertainty later.
- **Section 0 (Quick Facts)** is the operator-facing summary. Each line MUST be filled — never blank. Use `TBD` only as a last resort and only if the user explicitly chose to defer.
- **Sections 10–13 are mandatory** — if any is empty in the Korean draft, surface a final clarifying question before saving (per Step 3 validation gates in `SKILL.md`).
- Final line of the PRD must always be the "Run `/plan` next" pointer.
