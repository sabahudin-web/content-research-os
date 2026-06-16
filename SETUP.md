# Setup and Onboarding

This guide takes you from a fresh clone to a working `/content-research` command. Plan about 30 to 45
minutes for the first setup, most of it spent creating your Trigify searches and filling in your business
context.

Work top to bottom. Do not skip the context step, the system cannot score content without it.

---

## Step 0: What you need before you start

| Tool | Required? | Why | Where to get it |
|------|-----------|-----|-----------------|
| Claude Code | Yes | Runs the command | claude.com/claude-code |
| Trigify account | Yes | Scans the 5 social platforms | trigify.io |
| Trigify CLI | Yes | How the skill talks to Trigify | npm, see Step 2 |
| Notion | Yes | Where results are saved | notion.so |
| Fathom | Optional | Mines questions from your calls | fathom.video |
| LinkedIn fetch MCP | Optional | Fuller LinkedIn post bodies in Phase 2 | see Step 5 |
| yt-dlp | Optional | YouTube transcripts in Phase 2 | see Step 6 |

You can get a useful daily run with just **Trigify + Notion**. Add the rest later.

---

## Step 1: Install the repo

1. Clone this repository to your machine.
2. Open the folder in Claude Code.
3. Confirm Claude Code sees the command: type `/` and look for `content-research` in the list.

That is the whole install. The command and skills are already in `.claude/`.

---

## Step 2: Connect Trigify (the scanner)

Trigify is the engine that pulls posts from LinkedIn, Substack, YouTube, X, and Reddit. The skill talks to
it through a command-line tool.

1. Create a Trigify account and grab your API key from your Trigify settings.
2. Install the Trigify command-line tool (it is published on npm). In your terminal:
   ```
   npm install -g trigify
   ```
   If your provider ships it under a different package name, follow their install docs. The skill simply
   calls a `trigify` command, so any install that puts `trigify` on your PATH works.
3. Log in / set your key as the tool instructs (usually `trigify login` or setting an environment
   variable). Verify it works:
   ```
   trigify search list
   ```
   If that prints your searches (or an empty list) without an auth error, you are connected.

---

## Step 3: Connect Notion (where results land)

1. In your Claude client, connect the Notion integration (or set up the Notion MCP server). Copy
   `.mcp.json.example` to `.mcp.json` and fill in the Notion entry if your setup needs a token. `.mcp.json`
   is git-ignored, so your token never gets committed.
2. In Notion, create a database called **Content Research** with these three properties:
   - **Name** (title)
   - **Status** (select) with options: `Not Started`, `In Production`, `Done`
   - **Created time** (created time, auto)
3. Get the database ID and data source ID for that database. Then open
   `.claude/skills/acquisition-content-ideas/references/notion-content-research-db.md` and replace these
   placeholders with your real values:
   - `YOUR_NOTION_CONTENT_RESEARCH_DB_ID`
   - `YOUR_NOTION_DATA_SOURCE_ID`
   - `YOUR_NOTION_PARENT_PAGE_ID` (the page you want the daily run pages to live under)

   Tip: the database ID is the long string in the database URL. If you are unsure how to find the data
   source ID, ask Claude Code: "find the data source ID for my Content Research Notion database" once Notion
   is connected.

---

## Step 4: Create your Trigify searches and paste the IDs

This is the part that makes the system YOURS. You tell Trigify what topics to watch.

1. Open `.claude/skills/acquisition-content-ideas/references/trigify-search-ids.md`. It is a template with
   one row per search and a `YOUR_..._ID` placeholder in each.
2. Decide your 3 content pillars (you will also write these into `context/content-pillars.md` in Step 7).
3. For each row, create the matching Trigify search. The easiest way: ask Claude Code to run the
   `trigify-tools` skill and say what you want, for example: "create a LinkedIn keyword search for my pillar
   1 topics." It will create the search and hand you the ID.
4. Paste each returned ID into the matching row, replacing the placeholder.
5. You do not need every row filled to start. Even 3 or 4 good LinkedIn searches give a useful run. Add more
   over time.

The skill re-checks these IDs every run with `trigify search list` and updates the file if anything changed.

---

## Step 5: (Optional) LinkedIn post fetcher

In Phase 2, the system pulls the full body of winning LinkedIn posts. Most of the time Trigify already
returned the full text in Phase 1, so this is a fallback. If you want it:

1. Connect any MCP server that exposes a `get_linkedin_post_by_url` style tool.
2. Fill in the `linkedin-fetch` entry in `.mcp.json` with your provider's URL and auth.

If you skip this, Phase 2 still works. It uses whatever text Phase 1 captured and marks anything truncated.

---

## Step 6: (Optional) YouTube transcripts

For YouTube winners, Phase 2 grabs the transcript using `yt-dlp`, a free command-line tool.

1. Install it: `brew install yt-dlp` (Mac) or see the yt-dlp project for your OS.
2. Nothing else to configure. The skill calls it directly.

Videos 30 minutes or shorter get a full transcript. Longer videos get a 5-point summary to keep cost down.
Skip this and YouTube winners fall back to the video description.

---

## Step 7: (Optional) Fathom call-question mining

If you run sales or coaching calls and record them in Fathom, the system can turn the questions people ask
you into content ideas.

1. Connect Fathom in your Claude client, or fill in the `Fathom` entry in `.mcp.json`.
2. That is it. Phase 1 automatically checks the last 24 hours of calls. No calls found means it skips this
   cleanly, no error.

---

## Step 8: Fill in your business context (do not skip)

Every skill reads the files in `context/`. They ship blank. Until you fill them in, the system cannot tell
good content from noise for YOUR audience.

Open `context/README.md` for the guided tour. Fill them in this order:

1. **`context/content-pillars.md`**, the most important file. Your 3 pillars, your content mix, the
   "Uncopyable Core" checklist, and what you never post. The scoring engine lives or dies on this.
2. **`context/business-profile.md`**, who you are, what you sell, who you serve.
3. **`context/my-voice-dna.md`**, your tone and the words you do and do not use.
4. **`context/positioning.md`**, what makes you different.
5. **`context/proof-and-results.md`**, real outcomes you can cite. Only real ones, never invent numbers.
6. **`context/current-priorities.md`** and **`context/patterns.md`**, what matters now and how you like the
   assistant to work.
7. **`.claude/skills/acquisition-content-extraction/references/my-story-doc.md`**, a few real stories from
   your journey, used to suggest newsletter angles.

You do not have to make them perfect. Fill them honestly, run the system, and improve them as you see what
the winners look like.

---

## Step 9: First run

In Claude Code, type:

```
/content-research
```

It will scan, score, extract, and write to Notion end to end with no pauses. When it finishes you get a
3-line summary: the Notion page link with the winner count, the top winner per pillar, and a cost report.

Open the Notion parent page, read the winners, and flip the ones you want to produce from `Not Started` to
`In Production`.

---

## Step 10: (Optional) Run it every morning automatically

If your Claude client has a routines or scheduling panel, add `/content-research` to run Monday to Friday in
the morning. Then your winners are waiting for you before you sit down.

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `trigify search list` errors | Your Trigify CLI is not logged in. Re-run the login step in Step 2. |
| Run says "missing search" | A row in `trigify-search-ids.md` still has a placeholder ID. Fill it in or ignore, the run continues with the searches that exist. |
| Nothing written to Notion | Check the Notion connection and that the IDs in `notion-content-research-db.md` are real, not placeholders. The markdown file is still saved locally either way. |
| Scoring picks weird winners | Your `context/content-pillars.md` is thin or still a template. Sharpen the pillars and the Uncopyable Core checklist. |
| 0 winners | Normal on a quiet day, or your searches are too narrow. Widen the Trigify keywords. |
| YouTube winners have no transcript | Install `yt-dlp` (Step 6), or accept the description fallback. |

---

## Where things are saved

- Daily markdown: `inbox/outputs/md/`
- Notion: your Content Research database
- Temporary handoff between Phase 1 and Phase 2: `/tmp/phase1/` (safe to ignore)

You are set. Run `/content-research` and let it work.
