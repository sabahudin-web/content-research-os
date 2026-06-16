# Trigify Search IDs: Daily Content Research (TEMPLATE)

**Last updated:** TEMPLATE, fill this in after you create your own searches.

> This file is your search inventory. The `acquisition-content-ideas` skill reads it every run, then
> verifies the IDs are still live with `trigify search list` and updates this file if anything changed.
> Replace every `YOUR_..._ID` placeholder below with the real Trigify search ID you get back when you
> create the search. Never invent IDs. If a search does not exist yet, create it first (see SETUP.md),
> then paste its ID here.

Pillar mapping legend (rename these to YOUR three content pillars):
- **P1** = [Your Pillar 1, e.g. "the core system you teach"]
- **P2** = [Your Pillar 2, e.g. "the go-to-market angle"]
- **P3** = [Your Pillar 3, e.g. "the audience-reality / anti-hype angle"]
- **general** = topic-broad, not pillar-anchored (assigned by AI during scoring)

---

## How to create each search

For every row below, run the `trigify-tools` skill (or the Trigify CLI directly) to create a keyword or
profile search, then copy the returned ID into the `ID` column. Example:

```
trigify search create \
  --name "Daily Research - LinkedIn - [Your Pillar 1]" \
  --type linkedin-posts \
  --keywords "term a,term b,term c" \
  --frequency DAILY
```

The CLI prints the new search ID. Paste it into the matching row.

---

## LinkedIn Posts (4 searches: pillar-split + general)

| Name | ID | Pillar | Status | Last results |
|------|-----|--------|--------|--------------|
| Daily Research - LinkedIn - [Pillar 1 topic] | `YOUR_LI_P1_SEARCH_ID` | P1 | pending |, |
| Daily Research - LinkedIn - [broad topic] | `YOUR_LI_GENERAL_SEARCH_ID` | general | pending |, |
| Daily Research - LinkedIn - [Pillar 2 topic] | `YOUR_LI_P2_SEARCH_ID` | P2 | pending |, |
| Daily Research - LinkedIn - [Pillar 3 topic] | `YOUR_LI_P3_SEARCH_ID` | P3 | pending |, |

LinkedIn pull strategy: 50 to 100 posts total across all 4 searches per day. Rough split: 25 / 15 / 30 / 30.

---

## Substack Posts (1 search: general)

| Name | ID | Pillar | Status | Last results |
|------|-----|--------|--------|--------------|
| Daily Research - Substack - [broad topic] | `YOUR_SUBSTACK_SEARCH_ID` | general | pending |, |

**Known Trigify quirk:** `trigify search update` does not reliably update keyword fields on Substack
searches (only `--name` updates stick). If you need to change Substack keywords, delete and recreate the
search rather than updating it.

---

## YouTube: channel monitors + 1 keyword search

**Channel monitors (track specific creators in your niche):**

| Name | ID | Pillar | Status | Last results |
|------|-----|--------|--------|--------------|
| [creator-1]-yt | `YOUR_YT_CHANNEL_1_ID` | general | pending |, |
| [creator-2]-yt | `YOUR_YT_CHANNEL_2_ID` | general | pending |, |
| [creator-3]-yt | `YOUR_YT_CHANNEL_3_ID` | general | pending |, |

**Keyword search (1, captures rising videos from any creator):**

| Name | ID | Pillar | Status | Last results |
|------|-----|--------|--------|--------------|
| Daily Research - YouTube - [your topic] Tutorials | `YOUR_YT_KEYWORD_SEARCH_ID` | general | pending |, |

**Keyword config example:** OR `["your term", "your term 2", "your category"]` · AND `["tutorial"]` ·
NOT `["crypto", "trading", "web3"]` · past-week · max 20 · DAILY

YouTube pull strategy: `--limit 3` per channel monitor + `--limit 20` for the keyword search. **Hard cap of
3 YouTube winners post-scoring** (Phase 1 Step 5), because YouTube transcript extraction in Phase 2 is the
single most expensive operation in the run.

If a channel goes silent for weeks, pause it (move it out of the active table) so you do not burn credits on
0-result polls.

---

## X (Twitter): general + pillar searches + profile

| Name | ID | Type | Pillar | Status | Last results |
|------|-----|------|--------|--------|--------------|
| [your topic]-x-posts | `YOUR_X_GENERAL_SEARCH_ID` | twitter-posts (keyword) | general | pending |, |
| Daily Research - X - [Pillar 1 topic] | `YOUR_X_P1_SEARCH_ID` | twitter-posts (keyword) | P1 | pending |, |
| Daily Research - X - [Pillar 2 topic] | `YOUR_X_P2_SEARCH_ID` | twitter-posts (keyword) | P2 | pending |, |
| Daily Research - X - [Pillar 3 topic] | `YOUR_X_P3_SEARCH_ID` | twitter-posts (keyword) | P3 | pending |, |
| [key-creator-handle] | `YOUR_X_PROFILE_SEARCH_ID` | twitter-profile | general | pending |, |

**Pull strategy:** General + 3 pillar searches contribute 80 to 90 posts/day. A key-creator profile adds 5 to 10.
Total cap: 100.

---

## Reddit: 1 general keyword search

| Name | ID | Pillar | Status | Last results |
|------|-----|--------|--------|--------------|
| Daily Research - Reddit - [your topic] | `YOUR_REDDIT_SEARCH_ID` | general | pending |, |

**Keyword config example:** OR `["your term", "your category", "your audience term"]` · AND `["workflow"]` ·
NOT `["crypto", "nsfw", "airdrop"]` · past-24h · max 100 · DAILY

**Subreddit filtering happens during Phase 1 Step 4 (aggregation), not at search time**, Trigify
`reddit-posts` is keyword-based with no native subreddit filter. After pulling, keep only posts from the
subreddits relevant to your audience and drop the rest. Edit the keep-list inside the Phase 1 skill (Step 4)
to match your niche.

---

## After your first runs

The skill refreshes this file automatically each run via `trigify search list`. Once you have a few days of
data, the `Last results` column tells you which searches are producing winners and which to trim. If a
required search is still missing at run time, the skill logs the gap to its Cost Report and continues with
what is available, it never fabricates IDs.
