---
name: trigify-tools
description: MUST USE for ANY Trigify operation, searches, workflows, enrichment, X/Twitter, Jarvis, credits, testing, or anything involving social listening and workflow automation. This skill contains critical rules about testing procedures and search design that prevent common mistakes. Trigger this skill even for simple tasks like "test this workflow", "create a search", "check credits", or "send a test event". Also trigger when the user mentions social listening, keyword monitoring, track competitors, monitor brand, capture engagers, enrich profile, enrich company, post on X, signal detection, even if the task seems straightforward. Skipping this skill leads to fabricated test data and poorly designed searches.
---

# Trigify Tools Guide

Help users create effective social listening searches and automated workflows using Trigify.

## Core Philosophy

1. **Be a consultant, not an order taker.** Most users don't know what's possible with social listening. Guide them by suggesting signal types and use cases.
2. **Searches cast a broad net. Workflows extract specific signals.** Don't over-filter in the search, the AI agents in workflows do the precision work.
3. **Always look up examples first.** Before building a workflow, look up existing workflow examples to see proven patterns. Don't reinvent the wheel.
4. **Test with real data. NEVER fabricate test posts.** When testing workflows, ALWAYS pull actual posts from the attached search first, then use a real post as the test payload. Fabricated/dummy posts produce misleading test results.

---

## Capabilities at a Glance

| Category | Key Operations | When to Use |
|----------|---------------|-------------|
| **Searches** | Create, list, get results, update, delete | Setting up social listening for keywords or profiles |
| **Workflows** | Create, list examples, test, execute, modify | Automating actions on social listening data |
| **Enrichment** | Enrich profile, enrich company | Getting LinkedIn data (title, company, email) for a person or company |
| **X/Twitter** | Post, reply, like, repost, follow, DM, lookup user | Managing X accounts and engagement |
| **Jarvis AI** | Start conversation, send message, check task | Natural language workflow building |
| **Content** | Get post by URL, get comments, get profile posts | Analyzing specific posts or profiles |
| **Integrations** | List integrations, check health, get CRM fields | Checking connected tools (CRM, Slack, etc.) |
| **Credits** | Check balance, usage breakdown | Checking account usage and remaining credits |
| **Tracking** | Bulk track profiles, remove tracked profile | Monitoring specific LinkedIn/Twitter profiles |

---

## Part 1: Creating Effective Searches

A search is the foundation, it defines what content Trigify captures from social media. Getting this right determines everything downstream.

### Signal-Oriented Discovery

Before asking about keywords or platforms, understand the user's goal. Ask:

> "At a high level, are you looking to **generate leads/pipeline** or **gather market intelligence**?"

Then suggest specific signal types based on their answer.

**Sales/Growth signals:** Competitor Engagement Capture, Pain Point Discovery, Dissatisfied Customer Signals, High-Intent Topic Monitoring, Comparison Shopping, Funding Signals, Hiring Signals, Champion Movement Tracking.

**Marketing Intelligence signals:** Brand Mention Monitoring, Sentiment Shift Detection, Share of Voice, Content Trend Detection, Question Mining, Competitor Content Analysis, Feature Request Mining.

Once a signal type is chosen, explain:
> "I'll create a search that captures [category of content] broadly. Then we'll build a workflow with AI to extract [specific signal] from those results."

### Boolean Query Rules

**Read `references/search-guide.md` before creating any search**, it has the full Boolean query reference, platform-specific rules, and keyword strategy patterns. The critical rules summarized here:

- **6 keywords max** across OR + AND + NOT combined (except Instagram: 30 hashtags, OR only)
- **OR** = primary terms to capture (match any)
- **AND** = context to narrow (must also contain all)
- **NOT** = noise exclusion (exclude posts with any)
- **Don't over-filter**, use 2-3 OR terms, 1-2 AND for context, 1-2 NOT for noise
- **Split into multiple searches** when you need more precision (no downside, multiple searches can feed one workflow)

### Creating the Search

Key parameters when creating a search:
- `name`, descriptive name reflecting the signal type
- `keywords`, OR keywords (required for keyword searches)
- `keywordsAnd`, AND keywords (optional, for context)
- `keywordsNot`, NOT keywords (optional, for exclusion)
- `monitoringType`, platform + type (see reference for full list)
- `frequency`, how often to check (DAILY, WEEKLY, etc.)

**Keyword searches:** `linkedin-posts`, `twitter-posts`, `reddit-posts`, `youtube-videos`, `podcast-keywords`
**Profile monitoring:** `linkedin-profile`, `twitter-profile`, `youtube-channel`, `podcast-episodes`

### When to Use Profile vs Keyword Monitoring

- **Profile monitoring**: Track what a specific person/company posts ("monitor our CEO's LinkedIn")
- **Keyword monitoring**: Find posts about a topic across many authors ("find people talking about CRM pain points")
- **Brand monitoring**: Use BOTH, keyword search for what people say about you, profile search for your own posts

### Multi-Search Strategy

Proactively suggest splitting when:
- Tracking 2+ competitors → each gets its own focused search
- Need different context terms per topic
- Boolean logic needs > 6 keywords
- Different platforms need different keyword strategies

Frame it positively: "I'd recommend 2 searches, one for [X], one for [Y]. Both can feed the same workflow, but you'll get more targeted results."

---

## Part 2: Building Workflows

Workflows automate what happens when searches find content: enrich profiles, filter by ICP, send to CRM, notify Slack, etc.

### MANDATORY: Look Up Examples First

**Before creating any workflow, ALWAYS look up existing workflow examples.** This returns proven patterns you can adapt. Don't build from scratch when a tested example exists.

After reviewing examples, if building a custom workflow, look up all available action nodes to see what's possible.

### Workflow Patterns

**Read `references/workflow-patterns.md` before creating any workflow**, it has detailed patterns, decision trees, and examples. Quick reference:

| User Goal | Pattern | Key Components |
|-----------|---------|----------------|
| Enriched leads for outreach | Loop → Enrich → CRM | Loop, person enrichment, CRM action |
| Real-time alerts | Direct → Notify | Slack notification |
| Weekly summary | Scheduled → AI → Send | Fetch results, AI agent, Slack/email |
| Qualified leads only | Loop → Enrich → IF → CRM | Loop, enrichment, IF condition, CRM |
| Sentiment routing | Analyze → IF chain → Route | Sentiment agent, chained IF nodes |

### Trigger Selection

| User Intent | Trigger Type | Why |
|-------------|-------------|-----|
| Automate a single search | New Post | Simplest, fires per-post in real-time |
| Multiple searches, same logic | Multi-Post | Consolidation, deduplication |
| Batch summaries/digests | Scheduled | Use with fetch_search_results for date ranges |
| Chain workflows | Signal Created | Separate detection from action logic |
| External data | Webhook | CRM events, form submissions, Zapier/Make |

**Common mistake:** Using a real-time trigger for digests. "Summarize what competitors posted this week" = scheduled trigger + fetch, NOT a per-post trigger with AI on every post.

### Testing Workflows (CRITICAL)

**Always test before publishing. NEVER fabricate test data.**

#### For new-post triggered workflows (most common)

Every workflow with a new-post or multi-post trigger has a saved search attached to it. That search has already collected real posts. You MUST use one of those real posts as your test data. Here's the exact procedure:

1. **Get the workflow** to find which saved search is attached to it
2. **Get results from that saved search** to retrieve actual posts it has collected
3. **Pick a real post** from those results (ideally a recent one with engagement)
4. **Send that real post as the test payload**, pass the actual `text`, `authorUrl`, `postUrl`, `source`, `likes`, `comments`, `datePosted` fields from the real post
5. **Review the execution** to check each step passed with real data

**Why this matters:** Fabricated posts have fake author URLs that fail enrichment, fake text that produces meaningless AI analysis, and fake engagement numbers that skew filtering. Real posts test the entire pipeline end-to-end.

#### For scheduled workflows

Trigger the test normally, it simulates the scheduled execution and will pull its own data via fetch_search_results.

#### For webhook workflows

Provide a sample payload matching the expected schema.

#### After any test

Review the workflow execution to check each step's status. Verify enrichment returned data, AI analysis was meaningful, and outputs reached the destination.

### Credit-Saving Patterns

Credits matter, enrichment is where most get spent. Always filter BEFORE enriching:

**Bad:** Trigger → Person Enrichment → Email → IF (ICP check) → CRM
*(19 credits per post, even non-ICP contacts)*

**Good:** Trigger → Person Enrichment → IF (ICP check) → Email → CRM
*(4 credits for non-ICP, 19 only for qualified leads)*

**AI node selection:**
- Sentiment analysis → use the sentiment agent (2 credits, structured output)
- Custom analysis, scoring, summaries → use the general AI agent (variable credits)
- Always default to Claude Sonnet for AI agents, outperforms other options

---

## Part 3: Other Common Tasks

### Enrichment

- **Profile enrichment**, LinkedIn person data (name, title, company, location, email). Needs a LinkedIn URL.
- **Company enrichment**, Company data (size, industry, location). Needs a company name or domain.

### X/Twitter Management

Check connected accounts first. Then:
- Create posts, reply to tweets, like, repost, follow
- Send direct messages
- Look up user details by username

### Jarvis AI (Natural Language Workflows)

For users who prefer describing what they want in plain English:
1. Start a new Jarvis conversation
2. Describe the workflow in natural language
3. Poll for completion
4. Jarvis creates the search and/or workflow automatically

### Integration Health

Before building workflows that depend on integrations:
1. Check what's connected (list integrations)
2. Verify connections are working (health check)
3. Get field mappings for CRM fields, Slack channels, Notion databases, etc.

### Credits & Usage

- Check remaining credit balance
- View usage breakdown over time
- View per-feature breakdown
- 1 credit = 1 post monitored or 1 workflow action

---

## Reference Files (MUST READ when relevant)

These files are bundled with this skill. Read them before taking action, they contain critical details not covered above.

- **`references/search-guide.md`**, **Read before creating any search.** Contains Boolean query rules, platform-specific requirements (monitoring types, keyword limits, profile parameters), keyword strategy patterns per use case, and the multi-search splitting strategy.
- **`references/workflow-patterns.md`**, **Read before creating or modifying any workflow.** Contains decision trees for choosing triggers, enrichment nodes, AI nodes, and destinations. Includes full workflow pattern examples, signal-to-AI-prompt mapping, testing procedures, and the modification process for existing workflows.
