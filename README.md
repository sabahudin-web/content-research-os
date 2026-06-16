# Content Research for Claude Code

A daily content research engine that runs as a single Claude Code command: **`/content-research`**.

It scans 5 social platforms, scores every post against YOUR content pillars, and drops a short list of
production-ready content winners into your Notion workspace every morning. No more staring at a blank page
wondering what to post.

This repo is a template. Clone it, fill in a few files about your business, connect a couple of tools, and
you have the same content research system running for you.

---

## What you get

Run `/content-research` and the system:

1. Pulls 50 to 100 recent posts from **LinkedIn, Substack, YouTube, X, and Reddit** on the topics you care
   about (via Trigify).
2. Optionally mines the questions clients asked you on calls in the last 24 hours and turns them into
   content ideas (via Fathom, optional).
3. Scores every post against your 3 content pillars, your audience's pains, specificity, engagement, and
   timeliness, then keeps only the best (up to 25 winners).
4. For each winner, pulls the full source content plus a suggested way to repurpose it into a video, a
   newsletter, and a LinkedIn post.
5. Writes everything to a **Notion database**: one page for the day, one subpage per winner, ready to
   produce from.

The whole run is hands-off. You open Notion and pick what to make.

---

## What is inside this repo

| Folder | What it holds |
|--------|---------------|
| `.claude/commands/content-research.md` | The command itself. This is what `/content-research` runs. |
| `.claude/skills/acquisition-content-ideas/` | Phase 1: the scan and scoring engine. |
| `.claude/skills/acquisition-content-extraction/` | Phase 2: full context extraction for the winners. |
| `.claude/skills/trigify-tools/` | Helper for creating and managing your Trigify searches. |
| `.claude/rules/` | Small rules that load automatically (where outputs go, which context to read). |
| `context/` | The brain of the system. Blank templates about your business. **Fill these in first.** |
| `brain/content-moc.md` | A knowledge map that fills up with content lessons as you run the system. |
| `inbox/outputs/md/` | Where your daily research markdown files land. |

The skills use Claude Code's built-in parallel sub-agents to do the heavy lifting (5 platform scanners, then
up to 25 extractors, then 5 Notion writers, all running at once). You do not configure these. They are part
of the skills.

---

## The 30-second mental model

```
/content-research
      |
      v
  Phase 1  (acquisition-content-ideas)
   scan 5 platforms -> score -> pick <=25 winners -> write Notion parent page
      |
      v
  Phase 2  (acquisition-content-extraction)
   pull full content for each winner -> write a Notion subpage per winner
      |
      v
  You open Notion, pick what to produce
```

---

## Get started

1. **Install:** clone this repo and open it in Claude Code (see `SETUP.md`).
2. **Connect tools:** Trigify (required), Notion (required), Fathom and a LinkedIn fetcher (optional). Step
   by step in `SETUP.md`.
3. **Fill in your business:** complete the files in `context/`, starting with `context/content-pillars.md`.
   The `context/README.md` explains each one.
4. **Create your searches:** set up your Trigify searches and paste their IDs into
   `.claude/skills/acquisition-content-ideas/references/trigify-search-ids.md`. `SETUP.md` walks you through
   it.
5. **Run it:** type `/content-research` in Claude Code.

Full instructions are in **`SETUP.md`**. Read that next.

---

## Requirements at a glance

- **Claude Code** (the thing that runs the command).
- **Trigify account** plus the Trigify command-line tool (the social listening engine).
- **Notion** connected to Claude Code (where results are saved).
- **Fathom** (optional, for call-question mining).
- A **LinkedIn post fetch** MCP tool (optional, improves Phase 2 on LinkedIn winners).
- **yt-dlp** installed locally (optional, for YouTube transcripts).

You do not need all of these to start. Trigify plus Notion is enough for a useful daily run.

---

## A note on cost

This system is designed to keep cost predictable. The expensive step (pulling full post bodies and video
transcripts) runs ONLY on the winners, never on the hundreds of posts scanned. YouTube is hard-capped at 3
winners a day because transcripts are the most expensive operation. Each run ends with a Cost Report telling
you exactly where your Trigify credits and tokens went, so you can trim what is not pulling its weight.

---

## License

Use it, fork it, adapt it to your business. See `LICENSE` if present, otherwise treat this as provided
as-is, no warranty.
