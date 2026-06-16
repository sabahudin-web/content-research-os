# Notion "Content Research" Database: Reference

**Last updated:** 2026-05-13 (first wire-up to daily research engine)

This file is the single source of truth for the Notion database that captures every daily content research run. Both Phase 1 (`acquisition-content-ideas`) and Phase 2 (`acquisition-content-extraction`) write to it.

---

## Database identity

| Field | Value |
|-------|-------|
| Database name | Content Research |
| Database ID | `YOUR_NOTION_CONTENT_RESEARCH_DB_ID` |
| Database URL | [https://www.notion.so/YOUR_NOTION_CONTENT_RESEARCH_DB_ID](https://www.notion.so/YOUR_NOTION_CONTENT_RESEARCH_DB_ID) |
| Data source ID | `YOUR_NOTION_DATA_SOURCE_ID` |
| Data source URL | `collection://YOUR_NOTION_DATA_SOURCE_ID` |
| Parent page | 2nd-brain-OS (`YOUR_NOTION_PARENT_PAGE_ID`) |

---

## Schema (current: lean by design)

| Property | Type | Notes |
|----------|------|-------|
| **Name** | title | Used for both parent run pages AND child winner subpages |
| **Status** | select | Options: `Not Started`, `In Production`, `Done` |
| **Created time** | created_time | Auto-set by Notion, used as the run date filter |

Pillar (P1/P2/P3) and Type (Daily Run / Per-Winner Brief) are NOT separate properties, they live in the page body as headings. This keeps the schema simple and avoids Notion's select-option clutter when filtering.

---

## How Phase 1 writes the parent page

Called from `acquisition-content-ideas` Step 7.

```
mcp__claude_ai_Notion__notion-create-pages
  parent:
    type: data_source_id
    data_source_id: YOUR_NOTION_DATA_SOURCE_ID
  pages:
    - properties:
        Name: "Daily Research YYYY-MM-DD, N winners, top theme: [#1 theme]"
        Status: "Not Started"
      content: <full markdown body from inbox/outputs/md/YYYY-MM-DD-acquisition-content-ideas.md>
```

The returned page ID is written to `/tmp/phase1/notion-parent-page-id.txt` so Phase 2 can attach subpages.

---

## How Phase 2 writes the winner subpages

Called from `acquisition-content-extraction` Step 4. Five `general-purpose` sub-agents run in parallel, each creating 5 subpages.

```
mcp__claude_ai_Notion__notion-create-pages
  parent:
    type: page_id
    page_id: <parent page ID from /tmp/phase1/notion-parent-page-id.txt>
  pages:
    - properties:
        title: "Winner #N, <hook + angle, 1 line>"
      content: <full Phase 2 brief, hook, full post copy, repurpose path, engagement, content format>
    [... 4 more per sub-agent ...]
```

Subpages inherit the parent's location in Notion. They do not appear in the database root view, they're nested under the daily run page. To query winner-level signals across runs, page through parent pages and recurse into children.

---

## Example URL pattern

- Parent run page: `https://www.notion.so/EXAMPLE_PARENT_RUN_PAGE_ID` (created 2026-05-13)
- Winner subpages: nested under the parent, returned in the `pages[].url` field of the create-pages response

---

## Adding properties later (when needed)

If the user wants filterable Pillar / Type properties later:

```
mcp__claude_ai_Notion__notion-update-data-source
  data_source_id: YOUR_NOTION_DATA_SOURCE_ID
  add_properties:
    Pillar: multi_select(P1, P2, P3)
    Type: select(Daily Run, Per-Winner Brief)
```

Then update both SKILL.md files' Notion push steps to set those properties on create. The body-headings approach is the current default, fewer moving parts, same readability.

---

## Status workflow (manual or downstream-skill driven)

- **Not Started**, written by Phase 1 / Phase 2 push. Default state.
- **In Production**, the user moves it here when they pick a winner for the day's content output (manual today; future: downstream skill auto-moves when a YouTube ideation / newsletter repurpose runs on that winner).
- **Done**, winner has shipped (post published / video uploaded / newsletter sent).

Status transitions are NOT automated by Phase 1/2, those skills only set `Not Started`. Phase 3 production skills can update later.

---

## Known constraint: Substack search update bug

Trigify CLI does NOT reliably update Substack search keywords (see `brain/trigify-substack-search-update-keywords-frozen.md`). Unrelated to this Notion DB, but worth noting since the Substack feed into this database depends on that search.
