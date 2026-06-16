---
name: acquisition-content-ideas
description: >
  Daily Mon–Fri content research engine. Pulls 50–100 posts per platform from LinkedIn, Substack,
  YouTube, X and Reddit via Trigify, scores every post against the user's pillars + engagement signals,
  filters through the Uncopyable Core, and outputs WINNERS ONLY (max 10 P1 / 10 P2 / 5 P3). No full
  context extraction here, that runs in Phase 2 only on winners, which is what keeps daily cost
  predictable. Use this skill when the user says "daily content research", "what's trending today",
  "find content winners", "run the content scan", "scan all platforms", or "content ideas". Also
  triggers automatically as Step 1 of the /content routine.
---

# Content Ideas: Daily Research Engine (Phase 1)

This is the daily research engine. It scans 5 platforms in parallel, scores hundreds of posts against the user's 3 pillars and engagement quality, and hands a clean winners list to Phase 2 (`acquisition-content-extraction`). Phase 2 is where the expensive full-context extraction happens, never here. That split is what keeps per-run cost under control.

**Automation level:** AUTONOMOUS, runs end-to-end, no checkpoints. The /content routine and the daily Mon–Fri schedule both depend on this not stopping mid-flight.

**Mode detection:**
- **Routine mode**, called by /content. Hands winners file forward to `acquisition-content-extraction` automatically.
- **Standalone mode**, called by name. Same workflow. Saves output and stops (the user chooses whether to run Phase 2 manually).

---

## Step 1: Context load + refresh search inventory

Read in parallel:
- `context/business-profile.md`, who the user is, what they sell
- `context/current-priorities.md`, what matters this quarter
- `context/patterns.md`, how the user thinks and works
- `context/my-voice-dna.md`, voice calibration (used by scoring's specificity check)
- `context/content-pillars.md`, the 3 pillars, mix, Uncopyable Core, NOT-to-post list
- `brain/content-moc.md`, past content patterns (read the Quick Reference table first; full entries only if 200+ lines)
- `references/trigify-search-ids.md`, current Trigify search inventory

Then refresh the inventory: run `trigify search list --limit 100` and compare to `references/trigify-search-ids.md`. If anything changed (new search added, ID missing, status changed), update the file in place. Never invent IDs, if a required search is missing, log it to the Cost Report at the end and continue with what's available.

---

## Step 2: Topic universe (locked)

Topic universe for daily runs:
- Claude Code systems
- AI OS / Agent OS
- Second Brain (architecture, memory, context engineering)
- Claude Code for GTM (outreach, prospecting, CRM automation)

In standalone mode, the user can pass an additional topic via flag. Otherwise, the universe stays locked, drift increases cost without improving winners.

---

## Step 3: Per-platform pulls (5 sub-agents in parallel, one per platform, 50–100 posts/platform cap)

Spawn **5 `general-purpose` sub-agents in parallel**, one per platform: LinkedIn, Substack, YouTube, X, Reddit. Each sub-agent owns ALL searches for its platform and returns one normalized JSON array: `[{post_text, author_handle, post_url, likes, comments, date, platform, source_search_id, pillar_hint}]`.

Sub-agent fan-out is the rule, not Bash+Python aggregation. Minutes matter on the daily schedule, 5 parallel sub-agents complete platform pulls in <2 min regardless of credit cost. Each sub-agent receives:
- The search IDs + per-search pull limit for its platform (from `references/trigify-search-ids.md`)
- The platform's pillar mapping rules
- The output JSON schema

**Hard cap: 100 posts per platform per day. Never exceed.**

### LinkedIn: 4 existing searches (3 pillar + 1 general)

Pull from these searches (IDs in `references/trigify-search-ids.md`):

| Search | Pillar tag | Pull limit |
|--------|-----------|-----------|
| Weekly Research - Claude Code Systems | P1 | 25 |
| Weekly Research - Second Brain & AI OS | general | 15 |
| Weekly Research - Claude Code for GTM | P2 | 30 |
| Weekly Research - Solopreneur AI Overwhelm | P3 | 30 |

Total: 100 posts. Pillar hint inherited from source search.

### Substack: 1 general search (mirrors LinkedIn general approach, NOT pillar-split)

Pull from `subtscak-post-inspo` with `--limit 100`. If this search returns 0 results, log it to the Cost Report and surface to the user in the output. Pillar hint = unassigned (assigned during scoring).

### YouTube: 3 channel monitors + 1 keyword search

YouTube channels post 1–3 videos/week (not daily), so 0-results-on-some-days is expected behavior. The keyword search compensates by catching new creators publishing on the topic universe.

**Channel monitors (3, keep existing):**

| Channel | ID | Pull limit |
|---------|-----|-----------|
| nick-saraev-yt | `YOUR_YT_CHANNEL_1_ID` | 3 |
| chase-ai-yt-channel | `YOUR_YT_CHANNEL_2_ID` | 3 |
| nate-herk-yt-channel | `YOUR_YT_CHANNEL_3_ID` | 3 |

**Keyword search (1, new, captures rising videos from any creator):**

| Search | ID | Pull limit |
|--------|-----|-----------|
| Daily Research - YouTube - Claude Code & AI OS Tutorials | `YOUR_YT_KEYWORD_SEARCH_ID` | 20 |

Total raw YouTube pull: up to 29 videos. Channels that return 0 are noted in Cost Report. Pillar hint = unassigned for keyword search; channel pulls inherit "general."

**Grace Leung channel monitor is paused**, silent for weeks. The skill no longer reads from it (search ID `YOUR_PAUSED_YT_CHANNEL_ID` exists in Trigify but is excluded here). No credits burn on 0-result polls. If Grace starts publishing again, re-add to the table above.

**⚠ YouTube engagement note:** Trigify's `engagement` object for YouTube DOES NOT return view counts, only `comments`. Use the relaxed thresholds in Step 5 (1+ comment = score 2, 5+ comments = score 3).

**⚠ HARD CAP, max 3 YouTube winners per day.** After scoring in Step 5, no matter how many YouTube candidates pass the rubric, only the **top 3 YouTube items** become winners. This is the cost guard, YouTube transcript extraction in Phase 2 is the single most expensive operation in the daily run, so we cap exposure here.

**YouTube pillar bias:** Default to P1 (Claude Code 2nd Brain OS) and P2 (GTM with Claude Code). Promote a P3 (Solopreneur Reality) YouTube winner ONLY when the video is genuinely exceptional, story-driven, anti-hype, raw, not just a passing topic match.

### X (Twitter): existing profiles + general keyword search

Pull from:

| Search | Pull limit |
|--------|-----------|
| claude-code-x-posts (keyword, general) | 50 |
| boris-claude-code (profile) | 20 |

If pillar-split X keyword searches exist (P1, P2, P3, check live inventory), pull 10 each instead and adjust the keyword pull down. Total cap: 100. Pillar hint = unassigned for general; inherited for pillar searches.

### Reddit: 1 general keyword search (covers the topic universe)

Pull from the Reddit search if it exists. If no Reddit search exists, log to Cost Report and skip. Sub filtering happens during aggregation (Step 4), not at the search level. Trigify `reddit-posts` is keyword-based, no native subreddit filter.

**Sub filter, drop r/Entrepreneur only.** Earlier "keep-list" filtering of 8 specific subs dropped 75 of 80 Reddit posts on first run because most relevant AI traffic came from off-list subs. The keyword + topic-universe scoring in Step 5 handles relevance, let it work, don't pre-filter by sub.

**⚠ Reddit engagement note:** Trigify returns `likes: 0` for ALL Reddit posts (upvote counts not exposed). Engagement scoring for Reddit must rely entirely on comment count.

Pull limit: 100. Pillar hint = unassigned.

---

## Step 3.5: Pull call questions (Fathom MCP, additive bundle)

This step runs in parallel with Step 3 (sub-agent fan-out), kick it off as a 6th parallel call so it doesn't add to wall-clock time.

**Why this step exists:** Clients and prospects ask the user the SAME questions repeatedly on coaching/sales/nurturing calls. Those questions are content gold, they're real demand signal sourced from real conversations. Every daily run captures the last 24h of calls and turns them into one consolidated content idea bundle, additive to the 25 Trigify winners.

**This is NOT a duplicate of `operations-check-recent-calls`.** That skill reads commitments / next steps → Client Deliverables (different scope). This step reads QUESTIONS → content ideas. Different signal, different output, no shared logic.

### Execution

1. Call `mcp__claude_ai_Fathom__list_meetings` with parameters:
   - Filter to the last 24 hours
   - Include all call types (coaching, sales, nurturing, collaboration)
2. For each meeting returned (cap: 10 most recent), call `mcp__claude_ai_Fathom__get_meeting_summary` to fetch the structured summary. Summary is cheaper than transcript and usually surfaces the explicit questions section.
3. If summary lacks clear questions OR is missing, call `mcp__claude_ai_Fathom__get_meeting_transcript` for that single meeting and scan for question patterns:
   - "How do I [verb] [object]?"
   - "What's the best way to [action]?"
   - "Can Claude do [X]?"
   - "Why does [system] [behavior]?"
   - "Should I use [tool A] or [tool B]?"
   - Any sentence ending with `?` that the client/prospect (not the user) said
4. Consolidate into ONE bundle for the day. Cap at **5 questions max** to avoid bloat, if a single call surfaces 12 questions, keep the 5 most generalizable (a question that would apply to most solopreneur/SMB readers, not just the one client).
5. For each surviving question, propose **2–3 content angles** the user could take on it, short, specific, mapped to one of the 3 pillars.

### Skip behavior

- **No calls in last 24h:** skip cleanly. Mark in cost report. Do not block Phase 1.
- **Calls exist but no questions surface:** still output the section with a `Skip reason: no actionable questions this run` note. This tells the user the calls were dry, not that the step failed.
- **Fathom MCP auth fails:** log to cost report, skip. Do not retry.

### Scope guards

- This bundle does NOT count against the 25 Trigify winner cap (10 P1 / 10 P2 / 5 P3 still hold). Total per run = 25 + 1 bundle = up to 26 items.
- This bundle is NOT scored by the 5-criterion rubric. It bypasses scoring and always carries through to Phase 2 (when calls exist).
- The bundle is a single consolidated card with 2–5 question-cards inside it, not 5 separate winners.

---

## Step 4: Aggregate + dedupe

Merge all 5 platform sub-agent outputs into one normalized array. Dedupe by `post_url`. Apply the Reddit sub filter here (drop r/Entrepreneur only). Tag each post with `pillar_hint` (from source search where applicable) and `platform`.

The Fathom bundle from Step 3.5 is kept SEPARATE, it does NOT go into the Trigify aggregate pool, and it skips the scoring step entirely. It carries through to the output as its own section.

Expected aggregate size: 250–400 posts on a normal day.

---

## Step 5: CRUCIAL: Score → filter → pick winners (3 sub-agents in parallel)

This is the cost-sensitive step. First, engagement-prefilter the aggregate by dropping anything scoring 1 (below platform norm), this typically cuts the list from ~270 to ~60–80 candidates without an LLM call. Then **spawn 3 `general-purpose` sub-agents in parallel** to score the prefiltered candidates in chunks (~20–25 each). Three is the right number, engagement prefilter handles the bulk; LLM scoring only needs to evaluate Pillar fit + Pain match + Specificity + "Why now" on the survivors.

### Scoring rubric (1–3 per criterion, max 15 total)

| Criterion | 1 | 2 | 3 |
|-----------|---|---|---|
| **Pillar fit** | Doesn't map cleanly to any one pillar (blended or off-lane) | Maps loosely to one pillar | Maps cleanly to exactly ONE pillar (P1, P2, or P3) |
| **Pain match** | No clear pain | Generic pain | Hits a specific solopreneur/SMB-owner pain (AI noise, manual ops, 4-hour LinkedIn days) |
| **Specificity** | Abstract take | Some detail | Has a 5-second concrete moment / number / command / DM / file path |
| **Engagement quality** | Likes + comments below platform norm | Mid-tier engagement | Top-tier engagement vs platform norm (see thresholds below) |
| **"Why now" trigger** | No current tie-in | Loose tie-in | Tied to a current Claude update / AI hype moment / burnout cycle / community event |

**Engagement thresholds (post-level counts ONLY, never scrape reactors or commenters):**
- LinkedIn: 3 = >100 reactions OR >20 comments. 2 = 30–100 / 5–20. 1 = under.
- Substack: 3 = >50 likes OR >10 comments. 2 = 10–50 / 2–10. 1 = under.
- YouTube: **comments-only** (Trigify doesn't expose view counts). 3 = >5 comments. 2 = 1–5 comments. 1 = 0 comments.
- X: 3 = >500 likes OR >50 retweets. 2 = 100–500 / 10–50. 1 = under.
- Reddit: **comments-only** (Trigify returns likes: 0 for all Reddit posts). 3 = >20 comments. 2 = 5–20 comments. 1 = under.

**Drop everything under total score 10.**

### Pillar assignment (for posts with `pillar_hint = unassigned`)

The scoring subagent assigns a pillar based on content match:
- P1 (Claude Code 2nd Brain OS), system builds, routines, MCP, memory architecture, context engineering
- P2 (GTM with Claude Code), outreach, lead gen, DM automation, CRM, prospecting, intent
- P3 (Solopreneur Reality), burnout, FOMO, AI fatigue, comeback stories, anti-hype takes
- If the post blends two pillars, drop it (Pillar fit = 1 → total under 10 → auto-filtered).

### Uncopyable Core filter (read from `context/content-pillars.md`)

Run survivors through the Uncopyable Core Checklist (5 questions). Drop any post that fails 2+ checks.

### Apply pillar winner caps (priority order)

| Pillar | Max winners |
|--------|-------------|
| P1, Claude Code 2nd Brain OS | 10 |
| P2, GTM with Claude Code | 10 |
| P3, Solopreneur Reality | 5 |

**Total max: 25 winners.** If a pillar exceeds its cap after filtering, keep the top-scored items and drop the rest.

### Apply per-platform caps

YouTube has a separate hard cap that runs INSIDE the pillar caps above:

- **YouTube max: 3 winners total per day** (across all 3 pillars combined). This is the cost guard for Phase 2 transcript extraction, the single most expensive operation in the daily run.
- After scoring, sort YouTube candidates by total score. Keep top 3. Drop the rest. Then apply the pillar caps to the remaining list.
- YouTube pillar bias: prefer P1 + P2 winners. Promote a P3 YouTube winner ONLY when the video is genuinely exceptional (story-driven, anti-hype, raw), not on borderline topic match.

### Apply Perspective/Proof/Promo balance across the 25

Target across the full winners list: ~55% Perspective / ~30% Proof / ~15% Promo. If one category is over-represented, drop the lowest-score items from that category until balance holds (or as close as possible, never invent posts to pad).

### Topic frequency (content-gap signal)

Group all winners by topic theme. For each theme, count how many posts in the full aggregate list (Step 4 output) mentioned it. This frequency number is the content-gap signal, high-frequency themes with low winner-count = oversaturated; high-frequency with no winner = a gap the user can fill.

Save winners list with frequency tags. Format below.

---

## Step 6: Save output + cost report

Save to: `inbox/outputs/md/YYYY-MM-DD-acquisition-content-ideas.md` using the format below.

(Moved from old Step 7, happens BEFORE the Notion push so the markdown file always exists locally as the source of truth.)

---

## Step 7: Push parent page to Notion "Content Research" DB

This is MANDATORY in every daily run, the user uses both the MD and Notion paths for downstream production. Skip only if Notion is fully unreachable (and log the failure to cost report).

1. Read DB config from `references/notion-content-research-db.md` (DB ID, data source ID, schema).
2. Build the parent page payload:
   - **Title:** `Daily Research YYYY-MM-DD, N winners, top theme: [#1 frequency theme]`
   - **Status:** `Not Started`
   - **Body:** the full markdown from Step 6's output file (winners grouped by pillar with H2 / H3 headings, content gap signals, cost report)
3. Call `mcp__claude_ai_Notion__notion-create-pages` with parent = data_source_id, properties = `{Name, Status}`, content = body markdown.
4. Capture the parent page ID, it's the parent for the subpages in Phase 2 Step 4.
5. Write the parent page ID to a temp file at `/tmp/phase1/notion-parent-page-id.txt` so Phase 2 can pick it up.

If Notion push fails: log to cost report, continue (markdown file is the durable source of truth, the user can re-push manually).

---

## Step 8: Pass winners forward (no checkpoint)

In **routine mode**: immediately invoke `acquisition-content-extraction` (Phase 2). No "would you like me to proceed?" pause, daily Mon–Fri runs must complete end-to-end.

In **standalone mode**: tell the user: "Winners saved + Notion parent page created. Run `acquisition-content-extraction` to pull full context for production."

---

## Output reference: old Step 7 format kept below as the spec for Step 6's markdown

Save to: `inbox/outputs/md/YYYY-MM-DD-acquisition-content-ideas.md`

### Output format

```markdown
# Daily Content Research: Winners
**Date:** YYYY-MM-DD
**Aggregate pool:** [n] posts across 5 platforms
**Winners selected:** [n] (max 25, 10 P1 / 10 P2 / 5 P3)

---

## Pillar 1: Claude Code 2nd Brain OS ([n] winners)

### Winner #1: [hook + angle, 1 line]
- **Score:** [n]/15 · **Pillar:** P1
- **Source:** [platform] · [URL]
- **Engagement:** [likes] · [comments]
- **Trend / question:** [text] · **Frequency in pool:** [n] posts
- **Why it scored:** [1 sentence]

[repeat]

---

## Pillar 2: GTM with Claude Code ([n] winners)
[same format]

---

## Pillar 3: Solopreneur Reality ([n] winners)
[same format]

---

## Ideas from the calls (Fathom: last 24h)

- **Source:** [N calls reviewed: X coaching / Y sales / Z nurturing]
- **Window:** YYYY-MM-DD HH:MM → 24h prior
- **Skip reason:** [present only when no calls in window OR no actionable questions, then omit the rest of this section]

### Question #1: [verbatim question or close paraphrase]
- **Asked by:** [client name or "prospect (call name)"]
- **Call:** [Fathom recording URL]
- **Context:** [1-2 sentence framing of why the question came up]
- **the user's angle options (2-3):**
  - **Angle A (P[X]):** [content take in 1 sentence]
  - **Angle B (P[X]):** [content take in 1 sentence]
  - **Angle C (P[X]):** [optional]

[repeat per question, cap at 5 questions per bundle]

---

## Content Gap Signals (high-frequency, low-winner themes)
- [theme], [n] posts in pool, [n] winners → gap
- [theme], [n] posts in pool, [n] winners → saturated

---

## Cost Report
- Trigify credits used: [n]
- Posts pulled per platform: LinkedIn [n] / Substack [n] / YouTube [n] / X [n] / Reddit [n]
- Subagent invocations: [n] (pulls) + [n] (scoring) = [total]
- Approximate token spend: [tokens]
- Missing / failed searches: [list, e.g. "Reddit search not yet created"]
- Cut recommendation next run: [observation, e.g. "Substack returned 60 posts, only 1 winner, lower limit to 30"]
```

After the first 3 daily runs, append a 3-run rolling recommendation: which platform's pull limit can be reduced without sacrificing winner quality.

---

## Edge Cases

| Situation | Action |
|-----------|--------|
| `trigify search list` fails (auth, network) | STOP and surface the error to the user, do not fabricate inventory |
| A required search ID is missing | Log to Cost Report "missing: [name]", skip that platform, continue |
| A platform returns 0 posts | Note in Cost Report. Continue. Do not retry. |
| Fewer than 5 winners survive scoring | Lower the score floor from 10 to 8 ONCE, re-filter. If still under 5, output what's available. |
| One pillar produces 0 winners | Note in output. Do NOT invent winners to hit the cap. the user can request a manual deepen on that pillar via standalone mode. |
| Aggregate pool over 500 posts | The 100/platform cap should prevent this. If it happens, raise the score floor to 11 to avoid scoring-step token blow-up. |
| All winners cluster on 1–2 themes | Note in output as a saturation signal, the user may need to widen the topic universe. |
| Cost report shows credit spend > 200/day | Surface as a warning in the next morning's brief. The platform with the most credits used and lowest winner yield is the cut target. |

---

## Cross-area calls

- Reads: `context/`, `brain/content-moc.md`, `references/trigify-search-ids.md`, `references/notion-content-research-db.md`
- Calls: `trigify-tools` skill (for ID discovery + any search creation/update, never bypass), `mcp__claude_ai_Notion__notion-create-pages` (Step 7 parent page push), `mcp__claude_ai_Fathom__list_meetings` + `mcp__claude_ai_Fathom__get_meeting_summary` + `mcp__claude_ai_Fathom__get_meeting_transcript` (Step 3.5 call-questions bundle)
- Related skill (NOT reused, different scope): `operations-check-recent-calls` reads transcripts for commitments → Client Deliverables. This skill reads transcripts for QUESTIONS → content ideas. Both can coexist because they extract different signals from the same source.
- Hands forward to: `acquisition-content-extraction` (Phase 2) in routine mode
- Writes: `/tmp/phase1/notion-parent-page-id.txt` so Phase 2 can attach subpages to the right parent

---

## What good looks like

A winners file the user can open at 9 AM Mon–Fri and immediately see: which themes are hot today, which pillars are well-covered, which content gaps to fill. The list is short enough (max 25) to read in 2 minutes and rich enough (frequency tags + scores + engagement signals) to pick a production target without re-reading source posts. The Cost Report at the bottom shows whether yesterday's run was efficient, and exactly where to cut tomorrow's pull limits if cost is climbing. Phase 2 then takes only the winners forward, so the expensive context extraction never runs on dead leads.
