---
name: content-research
description: Daily content research engine. Runs Phase 1 (Trigify scan across 5 platforms + Fathom call questions + scoring → ≤25 winners) then Phase 2 (full context extraction + Notion parent page with 25 winner subpages). End-to-end, no checkpoints. Use when the user says "content research", "daily content scan", "what's trending today", "run the content research", or "find content winners".
allowed_tools:
  - Read
  - Write
  - Skill
  - Agent
  - Bash
---

# /content-research

You are the daily content research orchestrator. Run two skills in sequence, Phase 1 then Phase 2, and surface the Notion result. No checkpoints, no extra logic. Both skills do their own work end-to-end; this command just wires them together and reports back.

**Schedule:** To be set by the user via Claude Desktop Routines panel (target: Mon–Fri morning). Until scheduled, runs on demand.

**Automation:** AUTONOMOUS. Every step runs without user input. Phase 1 → Phase 2 → summary, no pauses.

**Context-first:** Before doing anything, read:
- `context/business-profile.md`
- `context/current-priorities.md`
- `context/patterns.md` (truth-teller / ruthless mentor directives)
- `context/my-voice-dna.md`
- `context/content-pillars.md`
- `brain/content-moc.md` (Quick Reference table first; full entries only if relevant)

---

## Workflow

Run these skills in order. Each skill receives the output of the previous one as context.

| # | Skill | Mode | What it does |
|---|-------|------|--------------|
| 1 | `acquisition-content-ideas` | AUTO | Phase 1. Refreshes Trigify search inventory, fans out 5 sub-agents (LinkedIn / Substack / YouTube / X / Reddit) to pull 50–100 posts/platform, runs Fathom MCP Step 3.5 to extract questions from the last 24h of calls, aggregates + dedupes, runs 3 scoring sub-agents on the prefiltered candidates, picks ≤25 winners (10 P1 / 10 P2 / 5 P3, YouTube hard-capped at 3), then pushes the parent page to Notion Content Research DB with: Run summary → Pillars 1/2/3 → Content Gap Signals table → Perspective/Proof/Promo balance → Cost Report (with Issues observed + Cut recommendations) → Phase 2 handoff. Adds an "Ideas from the calls" section to the parent page if Fathom returned questions. |
| 2 | `acquisition-content-extraction` | AUTO | Phase 2. Reads today's Phase 1 winners file + the parent page ID. Fans out up to 25 sub-agents in parallel, one per winner, to extract hook (verbatim) + full post copy or video transcript + content format + repurpose path (YouTube / Newsletter / LinkedIn). YouTube transcripts: full body for ≤30 min, sub-agent-generated 5-point summary for >30 min. Then 5 sub-agents in parallel each create 5 subpages via `notion-create-pages`, attaching all 25 as children of the Phase 1 parent page. Adds a conditional 26th subpage ("Ideas from the calls") when Phase 1 emitted call-question content. |

---

## After both skills complete

3. Surface a 3-line summary to the user:
   - Line 1: Notion parent page URL + total winners (P1 / P2 / P3 split) + top theme.
   - Line 2: Top winner per pillar, title only.
   - Line 3: Cost report highlights, Trigify credits used / sub-agent count / approximate token spend.
4. Run `operations-brain-ingest` if either phase surfaced a novel pattern (new platform behavior, a scoring edge case, a Trigify quirk, etc.). Skip if the run was uneventful.

---

## Output

- Phase 1 markdown: `inbox/outputs/md/YYYY-MM-DD-acquisition-content-ideas.md`
- Phase 2 markdown: `inbox/outputs/md/YYYY-MM-DD-acquisition-content-extraction.md`
- Notion Content Research DB (`YOUR_NOTION_CONTENT_RESEARCH_DB_ID`): parent page + ≤25 winner subpages (+ optional 26th "Ideas from the calls" page when Fathom found calls).

the user opens the Notion parent page, reviews the 25 winners, and flips Status from "Not Started" → "In Production" on the picks they want to produce. `/content-production` picks up from there.

---

## Edge cases

| Situation | Action |
|-----------|--------|
| Phase 1 produces 0 winners | Save Cost Report only, skip Phase 2, surface to the user with "no winners today, review aggregate pool" |
| Phase 1 fails mid-pull (Trigify auth, network) | Save partial output, surface error, do NOT proceed to Phase 2 |
| Phase 2 winners file missing after Phase 1 finished | Phase 1 failed silently, surface and stop |
| Notion push fails in either phase | Save the markdown files anyway, surface the Notion failure, do not retry more than once |
| Fathom MCP returns no calls in 24h | Phase 1 skips the "Ideas from the calls" section cleanly; Phase 2 skips the 26th subpage. No alert needed. |

---

## Scheduling

To be set by the user via Claude Desktop Routines panel, NOT CronCreate. Target cadence: Mon–Fri morning. Until scheduled, runs on demand.

---

## Rules

- Both skills are AUTONOMOUS, no pauses, no approval prompts. Run end-to-end.
- Phase 1 → Phase 2 hand-off is automatic. If Phase 1 fails, Phase 2 does NOT run.
- The Notion push happens INSIDE the skills (Phase 1 Step 7 = parent page; Phase 2 Step 4 = subpages as children). This command does NOT push to Notion separately.
- Sub-agent fan-out is preserved as-is: 5 platform sub-agents + 3 scoring sub-agents in Phase 1, up to 25 extraction sub-agents + 5 Notion push sub-agents in Phase 2.
- Hard cap on Phase 2: full-context extraction runs ONLY on winners, never on the long aggregate pool. That single rule keeps daily cost bounded.
- This command orchestrates, it does NOT contain skill logic.
