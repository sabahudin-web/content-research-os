# Context Loading Rule (content-research)

This rule auto-loads when Claude Code works inside `.claude/skills/`. It tells the skills which context
files to read so research and scoring reflect YOUR business, not a generic default.

## Always load (every skill, every run)

- `context/business-profile.md`, who you are, what you sell, who you serve
- `context/current-priorities.md`, what matters right now
- `context/patterns.md`, how you think, work, and want the assistant to behave

## Load for content work (the two content-research skills)

- `context/content-pillars.md`, your 3 pillars, content mix, the Uncopyable Core checklist, and the
  NOT-to-post list. The Phase 1 scoring engine reads this. If it is empty, scoring cannot work.
- `context/my-voice-dna.md`, voice calibration used by the specificity check during scoring
- `context/positioning.md`, your differentiation and trust layer
- `context/proof-and-results.md`, real outcomes the system may reference. Never fabricate these.
- `brain/content-moc.md`, accumulated content lessons. Read the Quick Reference table first.

If any context file is still a blank template, fill it before running. The system works only as well as
these files are filled in.

## Brain reading rule

`brain/content-moc.md` has a Quick Reference table at the top. Read that first. If the file is short, read
all of it. As it grows past a couple hundred lines, read only the Quick Reference plus the 2 to 3 entries
relevant to the current run. This keeps the cost flat as your brain grows.
