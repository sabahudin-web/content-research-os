# CLAUDE.md

This file tells Claude Code how to work in this repository. It loads automatically every session.

## What this repo is

This is a self-contained install of the **`/content-research`** command: a daily content research engine
that scans 5 social platforms, scores every post against your content pillars, and delivers a short list of
production-ready winners into your Notion workspace.

You (the person who installed this) run one command, `/content-research`, and the system does the rest.

## How the command works

`/content-research` runs two skills in sequence:

1. **Phase 1, `acquisition-content-ideas`** scans LinkedIn, Substack, YouTube, X, and Reddit through
   Trigify, optionally mines questions from your last 24h of calls (Fathom), scores everything, and picks up
   to 25 winners. It writes a parent page to your Notion Content Research database.
2. **Phase 2, `acquisition-content-extraction`** takes only those winners and pulls the full source
   content (post body, transcript, hook) plus a suggested repurpose path for video, newsletter, and
   LinkedIn. It writes one Notion subpage per winner under the Phase 1 parent page.

A third skill, **`trigify-tools`**, is a helper the system uses to create and manage your Trigify searches.

## Before anything runs

Every skill reads the files in `context/`. Those ship as blank templates. If they are still blank, the
system has no idea who you are or what to score against. **Fill in `context/` before your first run.** Start
with `context/content-pillars.md`, the most important one. See `SETUP.md` for the full checklist.

## Rules that load automatically

- `.claude/rules/context-load-by-skill.md`, which context files each skill reads
- `.claude/rules/inbox-routing.md`, where outputs get saved
- `.claude/rules/no-em-dashes.md`, an optional style rule you can keep or delete

## Output locations

- Phase 1 markdown: `inbox/outputs/md/YYYY-MM-DD-acquisition-content-ideas.md`
- Phase 2 markdown: `inbox/outputs/md/YYYY-MM-DD-acquisition-content-extraction.md`
- Notion: your Content Research database, parent page plus winner subpages

## What to do when something is missing

- A context file is still a template: stop and ask the user to fill it in.
- A Trigify search ID is missing: log it and continue with the searches that exist. Never invent an ID.
- Notion is unreachable: save the markdown files anyway, report the Notion failure, do not retry more than
  once.
