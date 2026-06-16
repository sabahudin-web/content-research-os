# Workflow Patterns Reference

## Table of Contents

1. [Trigger Decision Tree](#trigger-decision-tree)
2. [Enrichment Decision Tree](#enrichment-decision-tree)
3. [AI Node Decision Tree](#ai-node-decision-tree)
4. [Destination Decision Tree](#destination-decision-tree)
5. [Common Workflow Patterns](#common-workflow-patterns)
6. [Signal Type to AI Prompt Mapping](#signal-type-to-ai-prompt-mapping)
7. [Testing Workflows](#testing-workflows)
8. [Modifying Existing Workflows](#modifying-existing-workflows)

---

## Trigger Decision Tree

```
Real-time, per-post processing?
  → New Post (single search) or Multi-Post (multiple searches, same logic)

Batch summaries, digests, periodic reports?
  → Scheduled trigger + fetch_search_results

Chain workflows (detection → action)?
  → Workflow A creates signal → Workflow B uses Signal Created trigger

Data from outside Trigify?
  → Webhook trigger

Multiple searches, DIFFERENT logic per search?
  → Separate workflows with New Post trigger each (NOT multi-post)
```

## Enrichment Decision Tree

```
Have a LinkedIn URL, need person details?
  → person_enrichment (4 credits)

Need company info before spending on person enrichment?
  → company_enrichment (10 credits) or surfe_enrich_company (2 credits, cheaper)

Need email for outreach?
  → surfe_enrich_email (3 credits, 6-provider waterfall, higher accuracy)
  → email_enrichment (15 credits, native single provider, only if user specifically requests)

Need to find people at a company (no profile URL yet)?
  → surfe_find_people (1 credit/person)

Need phone number?
  → surfe_enrich_phone (26 credits), expensive, reserve for high-value targets
```

**Credit-saving order:** Always filter BEFORE enriching. Place cheapest filters first:
1. IF conditions (free)
2. Person enrichment (4 credits)
3. ICP check with IF (free)
4. Email enrichment (3-15 credits), only for qualified leads

## AI Node Decision Tree

```
Simple positive/negative/neutral classification?
  → get_sentiment (2 credits), structured output, plugs into IF conditions

Custom analysis, scoring, classification, summarization?
  → generic_agent (variable credits), flexible, any output format

Quick outreach copy?
  → copy_writer (5 credits), but most users prefer generic_agent for full control
```

**Model selection:** Always default to Claude Sonnet (anthropic/claude-sonnet-4.6). It outperforms other options even for simple tasks.

**Key insight:** For simple yes/no checks (e.g., "Is this a SaaS company?"), use generic_agent with boolean output + IF node. But don't use an agent when a plain IF on job title or company size works, that's wasted credits.

## Destination Decision Tree

```
Team visibility, immediate alerts?
  → Slack channel or Slack DM

Building a lead pipeline?
  → CRM: HubSpot or Attio (ask which they use)

Email outreach campaigns?
  → Instantly or Smartlead (requires email from enrichment)

LinkedIn outreach?
  → HeyReach (requires LinkedIn URL + firstName + lastName)

Multi-channel outreach?
  → La Growth Machine (requires both LinkedIn URL and email)

Internal typed alert for triage or chaining?
  → Signal (appears in Signals dashboard, can trigger other workflows)

Custom API / Clay / any system?
  → HTTP request (universal connector)

AI agent memory / cross-run context?
  → Save to DB (ONLY for agent memory, not business data)

Formal email / external recipients?
  → Gmail

Structured database for marketing ops?
  → Google Sheets, Airtable, or Notion (ask preference)
```

## Common Workflow Patterns

### Lead Gen / Engagement Tracking
```
Trigger: New post from search
→ Get post likers (linkedin_get_post_likes)
→ Loop through each liker
→ Enrich person (person_enrichment)
→ IF matches ICP (job title, company size, location)
  → TRUE: Get email → Push to CRM → Notify Slack
  → FALSE: Skip (loop continues)
→ Loop done: Exit
```

### Real-time Alerts
```
Trigger: New post from search
→ (Optional) Analyze sentiment
→ Send Slack message with post details
```

### Weekly Summary / Executive Report
```
Trigger: Scheduled (weekly)
→ Fetch recent posts (fetch_search_results, dateRange: week)
→ AI summarize the batch (generic_agent)
→ Send to Gmail and/or Slack
```

**Important:** Create SEPARATE reports per topic, brand mentions, competitor activity, and industry trends each get their own workflow. The AI produces better analysis when focused on one topic.

**Volume handling:** Under ~1,000 results/week → feed directly to agent. Thousands of results → use Loop + Agent Memory: loop processes each batch, agent saves to memory, then a final agent reads from memory to generate the consolidated report.

### Sentiment Router
```
Trigger: New post from search
→ Analyze sentiment
→ IF positive → Share to #wins channel
→ IF negative → Alert to #urgent channel
→ IF neutral → Log for trend analysis
```

### Qualified Lead Pipeline with Outreach
```
Trigger: New post from search
→ Loop through engagers
→ Person enrichment
→ IF ICP match
  → Email enrichment
  → IF email deliverable
    → Push to outreach tool (HeyReach/Instantly)
    → Notify sales rep via Slack DM
```

## Signal Type to AI Prompt Mapping

When the workflow includes an AI agent, the signal type determines the prompt:

### Sales/Growth Signals

| Signal Type | AI Prompt Focus |
|-------------|----------------|
| Competitor Engagement Capture | "Identify the person's role, company, and what the engagement suggests about their interest. Score as hot/warm/cold lead." |
| Pain Point Discovery | "Identify the specific pain point, severity (1-10), and whether our product could help. Extract the core frustration." |
| Dissatisfied Customer Signals | "Identify complaints, frustrations, or switching intent. Extract: what they're unhappy about, severity, alternatives mentioned." |
| High-Intent Topic Monitoring | "Identify if this is someone actively seeking recommendations. Extract: requirements, budget signals, timeline hints." |
| Comparison Shopping | "Identify which solutions they're comparing, what criteria matter, and any stated preferences." |

### Marketing Intelligence Signals

| Signal Type | AI Prompt Focus |
|-------------|----------------|
| Brand Mention Monitoring | "Classify sentiment (positive/neutral/negative). Extract: what they said, context, whether response is needed." |
| Sentiment Shift Detection | "Analyze sentiment vs typical baseline. Flag negative shifts with specific cause." |
| Content Trend Detection | "Identify if this represents an emerging trend. Extract: topic, velocity, who's talking about it." |
| Question Mining | "Extract the specific question, context, and whether we have content that answers it." |
| Feature Request Mining | "Identify the requested feature, use case behind it, and frequency signal." |

## Testing Workflows

### Before Testing

1. **Check integrations:** Verify all required integrations are connected and healthy
2. **Get integration data:** For Slack workflows, verify channels exist. For CRM, verify field mappings.

### Testing with Real Data (NEVER fabricate)

**For new-post triggered workflows, step by step:**

Every new-post/multi-post workflow has a saved search attached. That search has already collected real posts. Use them:

1. **Get the workflow details** to find which saved search is attached
2. **Get results from that saved search** to retrieve actual posts it has collected
3. **Pick a real post** from those results, one with engagement data (likes, comments, a real author URL)
4. **Send that real post as the test payload** passing the actual fields: `text`, `authorUrl`, `postUrl`, `source`, `likes`, `comments`, `datePosted`
5. **Review the execution** to verify each step worked with real data

**Why real data matters:** Fake posts have fake author URLs → enrichment fails or returns nothing. Fake text → AI analysis is meaningless. Fake engagement numbers → filtering logic isn't truly tested. The whole point of testing is to verify the pipeline works end-to-end with real content.

**For scheduled workflows:**
Trigger the test normally, it simulates the scheduled execution and pulls its own data.

**For webhook workflows:**
Provide a sample payload matching the expected schema.

### After Testing

1. Review the execution to see each step's status
2. Verify enrichment data came back correctly
3. Verify messages reached Slack/CRM/destination
4. Check credits consumed vs expected

## Modifying Existing Workflows

When a user wants to change an existing workflow:

1. **Fetch the current workflow** to see the live definition
2. **Check for existing drafts**, if a draft exists, base changes on it (it may have unsaved edits)
3. **Apply changes** to the workflow definition
4. **Save as a draft** before publishing
5. **Test the draft** (testing uses the draft automatically if one exists)
6. **Publish** after a successful test

**Key rules:**
- Always base changes on current workflow state, don't rebuild from scratch
- Preserve existing action IDs, changing them breaks version history
- Test after every modification before publishing
