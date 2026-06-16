# Search Creation Reference

## Boolean Field Structure

Each search has exactly THREE keyword fields. All keywords within a field are combined with that field's logic.

| Field | Logic | Purpose |
|-------|-------|---------|
| OR (`keywords`) | Match ANY | Primary terms to capture |
| AND (`keywordsAnd`) | Must ALSO contain ALL | Context to narrow results |
| NOT (`keywordsNot`) | Exclude ANY | Filter out noise |

**Total across all three fields: max 6 keywords** (except Instagram hashtags: 30, OR only).

You cannot have multiple AND groups or multiple OR groups. If you need more complex logic, split into multiple searches.

## Typical Keyword Structures

- 1 OR + 1 AND + 2 NOT = 4 keywords (focused, room to spare)
- 2 OR + 1 AND + 2 NOT = 5 keywords (good balance)
- 2 OR + 2 AND + 2 NOT = 6 keywords (maximum, fully used)

## Detecting Broad Keywords

**Red flags (too broad):**
- Single common words: "AI", "sales", "marketing", "growth", "tech", "data"
- Industry buzzwords without context: "automation", "cloud", "digital"
- Generic actions: "hiring", "looking for", "need help"
- Common words that are also brand names: "Clay", "Unify", "Notion"

**Good, specific keywords:**
- Unique brand names: "Trigify", "HubSpot", "Salesforce"
- Multi-word phrases: "sales enablement", "revenue operations"
- Specific use cases: "cold email templates", "LinkedIn automation"
- Problem statements: "CRM not syncing", "lead scoring broken"

**When keywords are broad:** Don't just create the search. Explain the issue and offer options:
1. User adds context keywords
2. Research the topic online and suggest keywords
3. Keep it broad (user accepts noise)

**Never assume context keywords.** If a keyword is generic (e.g., "Clay"), don't guess what AND/NOT terms to add. Ask the user.

## Boolean Strategy by Use Case

**Brand Monitoring:**
```
OR: ["CompanyName", "ProductName", "@handle"]
NOT: ["jobs", "hiring", "career"]
```

**Competitor Intelligence:**
```
OR: ["Competitor1", "Competitor2"]
AND: ["review", "alternative"]
NOT: ["jobs", "hiring"]
```

**Lead Generation (Pain Points):**
```
OR: ["struggling with", "hate my", "anyone recommend"]
AND: ["CRM", "sales tool"]
```

**Dissatisfied Competitor Customers:**
```
OR: ["CompetitorName"]
AND: ["alternative", "vs", "switched"]
NOT: ["jobs", "hiring"]
```
Note: Don't try to capture frustration with keywords like "hate" or "frustrated", the AI agent in the workflow is better at detecting sentiment than Boolean logic.

**Industry Trends:**
```
OR: ["social listening", "social monitoring"]
AND: ["trends", "report"]
```

## Multi-Search Strategy

Splitting into multiple searches is almost always better. There is NO downside, multiple searches can feed into a single workflow via the Multi-Post trigger.

**When to split:**
- Tracking 2+ competitors (each gets focused AND/NOT terms)
- Brand monitoring + competitor tracking (separate concerns)
- Need > 6 keywords for precision
- Different context terms per topic

**Example, Competitor Tracking:**

Instead of one search: `OR: ["Competitor1", "Competitor2", "review", "alternative"]` (4 keywords, no room for NOT)

Create two:
- Search 1: `OR: ["Competitor1"] AND: ["review", "alternative"] NOT: ["jobs"]`
- Search 2: `OR: ["Competitor2"] AND: ["review", "alternative"] NOT: ["jobs"]`

Both trigger the same workflow. Result: more precise, less noise.

## Platform-Specific Rules

### Keyword Searches

| Platform | monitoring_type | Keyword Limit | Boolean |
|----------|-----------------|---------------|---------|
| LinkedIn Posts | `linkedin-posts` | 6 total | OR + AND + NOT |
| Twitter/X Posts | `twitter-posts` | 6 total | OR + AND + NOT |
| Reddit | `reddit-posts` | 6 total | OR + AND + NOT |
| YouTube Videos | `youtube-videos` | 6 total | OR + AND + NOT |
| Podcasts | `podcast-keywords` | 6 total | OR + AND + NOT |
| Instagram Hashtags | `instagram-hashtag` | 30 hashtags | OR only |

### Profile Monitoring

| Platform | monitoring_type | Key Parameter |
|----------|-----------------|---------------|
| LinkedIn Profile | `linkedin-profile` | `profileUrl` |
| Twitter Profile | `twitter-profile` | `twitterProfileUrl` or `twitterUserId` |
| YouTube Channel | `youtube-channel` | `channelUrl` |
| Podcast Episodes | `podcast-episodes` | `podcastId` |
| Instagram Profile | `instagram-profile` | `instagramUsername` |

### Common Options (All Platforms)

- `frequency`, DAILY, WEEKLY, MONTHLY, QUARTERLY
- `maxResults`, 1-100 (default 50)
- `timeFrame`, past-24h, past-week, past-month, all-time

### LinkedIn-Specific Options

- `jobTitles`, Filter by job title (array)
- `contentType`, "videos", "photos", "documents", etc.
- `linkedinSortBy`, "date_posted" or "relevance"

### Instagram-Specific Rules

- 30 hashtags maximum, OR logic only (no AND/NOT)
- Hashtags entered without the # symbol
- `instagramSortBy`, "top" or "recent"

## Profile vs Keyword: When to Use Which

**Profile monitoring** when:
- Track a specific person's or company's posts
- "Monitor what our CEO posts"
- "Track competitor's official account"
- Goal is "see everything they post"

**Keyword monitoring** when:
- Find posts ABOUT a topic across many authors
- Catch mentions by anyone
- Goal is content discovery

**Brand monitoring, two-search strategy:**
One keyword search to catch what people say about the brand, plus one profile search to track the brand's own posts.
