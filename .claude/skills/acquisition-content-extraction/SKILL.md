---
name: acquisition-content-extraction
description: >
  Phase 2 of the daily content research engine. Reads today's WINNERS file from
  acquisition-content-ideas (Phase 1) and pulls FULL CONTEXT, verbatim hook, full post copy / video
  transcript / thread copy, and observed content format, ONLY for those winners (max 25). This is
  the expensive step, which is why it never runs on the long list. Output is production-ready
  context per winner: hook + pillar + trend evidence + repurpose path + full source content. Use
  this skill when the user says "extract content context", "build winner briefs", "pull full posts
  for winners", "production-ready context", or "extract content ideas". Also triggers automatically
  as Step 1.5 of the /content routine, after acquisition-content-ideas and before
  acquisition-content-youtube-ideation.
---

# Content Extraction: Full Context for Winners (Phase 2)

This is the cost-sensitive half of the daily research engine. Phase 1 (`acquisition-content-ideas`) does the wide scan and picks ≤25 winners. This skill takes that short list and pulls the FULL source content, hook verbatim, full post body or video transcript, and observed format, so downstream production skills (YouTube ideation, newsletter repurpose, LinkedIn repurpose) get production-ready briefs without re-fetching anything.

**Critical:** Never run full-context extraction on the long aggregate list, only on the winners file. That single rule is what keeps per-day cost bounded.

**Automation level:** AUTONOMOUS, runs end-to-end. No checkpoints. Pairs with Phase 1's daily Mon–Fri cadence.

**Mode detection:**
- **Routine mode**, called by Phase 1 or by /content. Reads `inbox/outputs/md/YYYY-MM-DD-acquisition-content-ideas.md` directly.
- **Standalone mode**, called by name. Same workflow. If no winners file for today, ask the user to run Phase 1 first OR paste a winners list manually.

---

## Step 1: Context load + winners check

Read in parallel:
- `context/content-pillars.md`, pillars + Uncopyable Core (used to validate repurpose paths)
- `context/my-voice-dna.md`, voice calibration (used when writing the repurpose-path suggestions)
- Today's winners file: `inbox/outputs/md/YYYY-MM-DD-acquisition-content-ideas.md`
- `references/my-story-doc.md (this skill's own copy)`, the user's personal stories (used for P3 newsletter angle matching)

If the winners file doesn't exist:
- **Routine mode:** STOP and surface to the user, Phase 1 must have failed silently.
- **Standalone mode:** ask the user to paste a winners list OR run Phase 1 first.

Extract the winners list. Expect ≤25 items, tagged by pillar (P1 / P2 / P3) with score, source URL, platform, engagement counts, and trend/frequency tag.

---

## Step 2: Per-winner full context extraction (one sub-agent per winner, mandatory parallel fan-out)

Spawn one `general-purpose` sub-agent per winner, up to 25 sub-agents in parallel. **Sub-agent fan-out is MANDATORY** even when full text is already in the Phase 1 aggregate JSON. the user's preference: minutes matter on daily schedule, so parallel sub-agents complete the full run in <2 min regardless of raw token cost. Bash+Python aggregation is not an acceptable substitute.

Each sub-agent receives the winner's metadata from Phase 1 and is responsible for assembling the FULL output record:
1. **First check `/tmp/phase1/aggregate_full.json`** for the post by URL, if found and the text field is ≥ 200 chars, USE THAT TEXT and skip external fetch. This saves the LinkedIn MCP / WebFetch call entirely (Trigify already returned the full body in Phase 1 for most posts).
2. **Only re-fetch externally** when the Phase 1 text is truncated or missing, then use the right tool for the platform:

### Tool routing by platform

| Platform | Tool | Notes |
|----------|------|-------|
| LinkedIn | `mcp__claude_ai_Relay_APP_Claude_Code__get_linkedin_post_by_url` | Returns full post body + engagement metadata. Single call per winner. |
| X (Twitter) | `trigify search results` already returned full post text, re-use; if thread, `WebFetch` the URL for the thread | No second Trigify call needed for single posts |
| Reddit | Phase 1 already returned full post text via Trigify | No second call. Use `WebFetch` on URL only if Phase 1 truncated. |
| Substack | `WebFetch` on the post URL | Returns full article body. Strip nav/footer chrome. |
| YouTube | `yt-dlp` via Bash for transcript + duration. Branch on duration. | Detailed rule below. |

### YouTube transcript length rule (cost guard)

YouTube transcripts are the single most expensive Phase 2 operation. Phase 1 caps YouTube at 3 winners; Phase 2 caps the transcript footprint per winner using this rule:

| Video duration | Extraction behavior |
|----------------|---------------------|
| ≤ 30 minutes | **Full transcript**, paste verbatim into `full_content` (same treatment as LinkedIn / X / Substack post copy). |
| > 30 minutes | **5-bullet structured summary**, spawn one extra `general-purpose` sub-agent to read the transcript and return: 1 hook, 5 key points (each 1-2 sentences max, capturing the actual claim/insight), 1 the user-angle take. The summary goes into `full_content` with a `[summary of N-minute video]` tag at the top. Full raw transcript is NOT saved. |

For both branches, also capture: video title (as hook), publish date, view count if available, channel name. If `yt-dlp` cannot fetch a transcript (auto-captions disabled / age-gated / region-locked), mark `status: "unverified"`, paste the video description from `WebFetch` instead, note in cost report. Do not retry more than once.

The 30-min threshold balances information density vs token cost, tutorial videos are typically 10–30 min (full transcript valuable), workshops/talks are 30–90 min (summary captures the claim without 50K-token paste).

Each subagent returns a normalized JSON object:
```json
{
  "winner_id": "N",
  "title": "hook + angle, 1 line",
  "pillar": "P1 | P2 | P3",
  "trend_or_question": "string",
  "frequency_in_pool": <int>,
  "platform": "linkedin | substack | youtube | x | reddit",
  "source_url": "string",
  "engagement": { "likes": <int>, "comments": <int> },
  "content_format": "text | image | carousel | video | thread",
  "hook_verbatim": "string, first 1-2 lines of the source post",
  "full_content": "string, full post body or transcript",
  "repurpose_path": {
    "youtube": "string, video angle in 1 sentence",
    "newsletter": "string, story match (from my-story-doc) + angle in 1 sentence",
    "linkedin": "string, post type recommendation (story | lead-magnet | infographic | carousel | contrarian | funny) + why in 1 sentence"
  },
  "status": "ok | unverified | failed"
}
```

### Cost guards

- **Phase 1 aggregate JSON reuse first.** Before any external fetch, check `/tmp/phase1/aggregate_full.json` for the post by URL. If text is ≥200 chars, use it directly. This eliminates most LinkedIn / Substack / X / Reddit re-fetches.
- **One retry max per external fetch.** If a transcript or post fetch fails twice, mark `status: "unverified"` and continue. Do not block the whole run on one stuck winner.
- **No reactor/commenter scraping.** Engagement = post-level counts only (already in Phase 1 output).
- **No re-scoring.** Pillar and score are inherited from Phase 1. This step extracts, it does not re-evaluate.
- **Paywalled Substack posts**, fetch what's visible above the paywall, mark `status: "unverified"`, note paywall in the output.
- **Deleted/removed posts (404 between Phase 1 and Phase 2)**, mark `status: "failed"`, note in cost report, drop from final output.

### Repurpose path generation rules

For each winner, the subagent proposes:
- **YouTube angle**, what's the core video concept the user could film? 1 sentence. Format type (Tutorial / System breakdown / Story / Contrarian / Comparison).
- **Newsletter angle**, which story from `my-story-doc.md` connects most naturally? 1 sentence bridge between the story and the idea. For P3 winners, the story match is mandatory. For P1/P2, optional but encouraged.
- **LinkedIn post type**, which of the 6 LinkedIn types (Story / Lead Magnet / Infographic / Carousel / Contrarian / Funny) fits best? 1 sentence why.

These are recommendations, not commitments, they help downstream skills (`acquisition-content-youtube-ideation`, `acquisition-content-newsletter-repurpose`, LinkedIn repurpose skills) start with a strong default.

---

## Step 3: Aggregate + write output

Collect all subagent outputs into a single report. Order by pillar (P1 → P2 → P3), then by Phase 1 score within each pillar.

### Output format

```markdown
# Content Extraction: Production-Ready Winners
**Date:** YYYY-MM-DD
**Winners processed:** [n] (from inbox/outputs/md/YYYY-MM-DD-acquisition-content-ideas.md)
**Status breakdown:** OK [n] · Unverified [n] · Failed [n]

---

## Pillar 1: Claude Code 2nd Brain OS ([n] winners)

### Winner #1: [hook + angle, 1 line]

**Pillar:** P1  |  **Phase 1 score:** [n]/15  |  **Status:** OK
**Trend / question answered:** [text]  |  **Frequency in pool:** [n] posts
**Source:** [platform] · [URL]
**Engagement:** [likes] likes · [comments] comments
**Content format:** [text | image | carousel | video | thread]

**Repurpose path:**
- **YouTube:** [angle] · Format: [Tutorial | Story | Contrarian | ...]
- **Newsletter:** [story match from my-story-doc] · Angle: [bridge sentence]
- **LinkedIn:** [post type] · Why: [1 sentence]

#### Full Context

**Hook (verbatim):**
> [first 1-2 lines of source post]

**Full post copy / transcript:**
```
[full text, paste exactly, preserve line breaks and emoji]
```

---

[repeat for each winner in pillar]

---

## Pillar 2: GTM with Claude Code ([n] winners)
[same format]

---

## Pillar 3: Solopreneur Reality ([n] winners)
[same format]

---

## Cost Report
- Subagents spawned: [n] (one per winner)
- LinkedIn `get_post_by_url` calls: [n]
- WebFetch calls: [n] (Substack [n] / YouTube [n] / X threads [n])
- yt-dlp transcript pulls: [n]
- Approximate token spend: [tokens]
- Failed fetches: [list, URL + reason]
- Unverified fetches: [list, URL + reason]
- Cut recommendation next run: [observation, e.g. "3 YouTube transcripts failed, consider dropping YouTube from Phase 1 winner pool unless transcripts available"]

---

## Handoff

Production-ready context for downstream skills:
- `acquisition-content-youtube-ideation`, anchors on top P1 or P2 winner with repurpose_path.youtube
- `acquisition-content-newsletter-repurpose`, anchors on top P3 winner with repurpose_path.newsletter
- LinkedIn repurpose skills (`acquisition-repurpose-linkedin-story`, `-contrarian`, `-carousel`, etc.), anchored on repurpose_path.linkedin
```

Save to: `inbox/outputs/md/YYYY-MM-DD-acquisition-content-extraction.md`

---

## Step 4: Push winner subpages to Notion (mandatory, parallel sub-agents + 1 conditional call subpage)

This is MANDATORY in every daily run, the user uses Notion subpages for downstream Phase 3 (long-form + short-form repurpose production).

1. Read the parent page ID from `/tmp/phase1/notion-parent-page-id.txt` (written by Phase 1 Step 7).
2. Read `references/notion-content-research-db.md` to confirm DB ID + schema is current.
3. Build 25 subpage payloads, one per Trigify winner. Each subpage has:
   - **Title:** `Winner #N, [hook + angle, 1 line]`
   - **Status:** `Not Started`
   - **Body (markdown):** the full winner brief from the extraction markdown, hook verbatim, full post copy, repurpose path (YouTube + Newsletter + LinkedIn), Phase 1 score, trend/frequency, engagement, content format
4. **Spawn 5 `general-purpose` sub-agents in parallel.** Each sub-agent creates 5 subpages via a single `mcp__claude_ai_Notion__notion-create-pages` call with `parent: {page_id: <parent_id>}` and a `pages: [...]` array of 5 payloads.
5. Aggregate sub-agent results: count subpages created, list any failures, capture each subpage URL.

### Step 4b: Conditional 26th subpage: "Ideas from the calls"

AFTER the 5 sub-agents finish (not in parallel, runs from the main thread), check the Phase 1 output markdown for an "Ideas from the calls" section.

- **If section exists AND is NOT skip-only** (i.e. it contains at least one Question card with angles): create ONE additional Notion subpage from the main thread via a single `mcp__claude_ai_Notion__notion-create-pages` call:
  - **Title:** `Ideas from the calls, YYYY-MM-DD`
  - **Status:** `Not Started`
  - **Parent:** the same daily run parent page ID
  - **Body:** the full "Ideas from the calls" markdown section verbatim from Phase 1, preserves every Question card, asked-by, call URL, context, and angle options
- **If section is absent OR has only a Skip reason note:** skip creation. Log to cost report: "No call-questions bundle this run."

This is the 26th subpage (or it's skipped on no-call days). It uses one main-thread call, NOT a 6th sub-agent, because it's a single page, and a sub-agent for one call is wasted setup cost.

If parent page ID is missing (Phase 1 Notion push failed): log to cost report, skip both the 25 winners AND the call-ideas subpage, but still write the local MD file.

---

## Step 5: Handoff (no checkpoint)

In **routine mode**: write the file, briefly summarize to the user ("25 winners extracted, top P1 = [title], top P2 = [title], top P3 = [title] · Notion parent: [URL] · 25 subpages created"), then hand control back to /content for Phase 3 (production: YouTube → newsletter → LinkedIn).

In **standalone mode**: same summary, then tell the user: "Ready for production. Run `acquisition-content-youtube-ideation` to pick the video target, or jump straight to a LinkedIn repurpose skill with one of the winners as input. Notion subpages available at [parent URL]."

---

## Edge Cases

| Situation | Action |
|-----------|--------|
| No winners file for today | Routine mode → STOP and surface (Phase 1 failed). Standalone → ask the user to run Phase 1 or paste a winners list. |
| Winners file has 0 winners | Output a one-line note: "Phase 1 produced no winners today, review aggregate pool in Phase 1 cost report." Skip extraction. |
| LinkedIn `get_post_by_url` returns 403 / rate limit | One retry. Then `status: "unverified"`, paste the snippet Phase 1 already captured, note in cost report. |
| YouTube transcript unavailable | Mark `[transcript unavailable]` in full_content, paste the video description from `WebFetch` instead, `status: "unverified"`. |
| Substack post paywalled | Capture above-paywall content, mark `status: "unverified"`, note paywall in cost report. |
| Post deleted between Phase 1 and Phase 2 (404) | `status: "failed"`. Drop from final output. List in cost report. |
| One pillar has 0 winners (Phase 1 result) | Skip that pillar's section entirely. Note "0 winners this run" in the header. |
| All 25 winners fail extraction | Output cost report only with failure analysis. Surface as a hard alert, likely a tool outage. |
| Token spend climbing past comfortable threshold | After 3 daily runs, surface a recommendation: which platform's winners produce the lowest yield-to-cost ratio (e.g. "YouTube transcripts cost 2x other platforms, consider lowering YouTube pull in Phase 1") |

---

## Cross-area calls

- Reads: today's `acquisition-content-ideas` output, `/tmp/phase1/aggregate_full.json`, `/tmp/phase1/notion-parent-page-id.txt`, `context/content-pillars.md`, `context/my-voice-dna.md`, `my-story-doc.md`, `.claude/skills/acquisition-content-ideas/references/notion-content-research-db.md`
- Tools used: `mcp__claude_ai_Relay_APP_Claude_Code__get_linkedin_post_by_url`, `WebFetch`, `Bash` (for yt-dlp transcripts), `trigify-tools` skill (only if re-fetching is required), `mcp__claude_ai_Notion__notion-create-pages` (Step 4 subpage push, 25 Trigify winners via 5 parallel sub-agents + 1 conditional "Ideas from the calls" subpage from main thread)
- Hands forward to: `acquisition-content-youtube-ideation` (Step 2 of /content), `acquisition-content-newsletter-repurpose`, and LinkedIn repurpose skills

---

## What good looks like

the user opens today's extraction file at 9 AM Mon–Fri and gets up to 25 production-ready briefs, hook verbatim, full source text, content format observed, and a recommended repurpose path for video + newsletter + LinkedIn. They can hand any single brief to a production skill and ship without re-reading the source. The Cost Report tells them which platform's full-context pulls were expensive (transcripts, paywalls, retries) so the next day's Phase 1 can adjust pull limits intelligently, high-cost low-yield platforms get trimmed, high-yield platforms stay or expand. Over a week, the cost-to-winners ratio per platform becomes visible, and pull limits self-tune.
