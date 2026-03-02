---
name: reddit-monitor
description: "Hourly scan of target subreddits for posts about precious metal tracking apps, spot price tools, and portfolio spreadsheets."
metadata:
  {
    "openclaw":
      {
        "emoji": "🔍",
        "requires": { "bins": ["curl", "jq"] }
      }
  }
---

# Reddit Monitor — Precious Metals App & Tool Mentions

Scans target subreddits hourly for posts related to precious metal tracking apps, spot price apps, portfolio trackers, and spreadsheets. Returns structured JSON for the agent to act on (engage, analyze, report).

## Target Subreddits

Read from MEMORY.md. Current list:

- r/Silverbugs
- r/Pmsforsale
- r/CoinSales
- r/Bullion
- r/PreciousMetals
- r/Silver

If the user updates the subreddit list in MEMORY.md, use the updated list.

## Search Keywords

Use these terms when searching each subreddit. Combine with OR logic by running one search per keyword group:

**App/tool mentions:**
- `app OR application OR mobile app`
- `track OR tracker OR tracking OR portfolio`
- `spreadsheet OR google sheets OR excel`
- `spot price OR live price OR price alert`
- `stack tracker OR stack value OR stack worth`
- `inventory OR catalog OR collection manager`

**Competitor/alternative mentions:**
- `stackerscan OR stacker scan`
- `apmex app OR jm bullion app OR sd bullion`
- `goldprice OR kitco OR bullionvault OR goldfolio`
- `precious metals app OR silver app OR gold app`

## How to Run the Scan

For each subreddit and keyword group, run:

```bash
curl -s -H "User-Agent: openclaw-agent/1.0" \
  "https://www.reddit.com/r/SUBREDDIT/search.json?q=KEYWORDS&restrict_sr=on&sort=new&t=hour&limit=25" \
  | jq '[.data.children[] | {
    id: .data.id,
    subreddit: .data.subreddit,
    title: .data.title,
    author: .data.author,
    selftext: (.data.selftext | if length > 300 then .[0:300] + "..." else . end),
    url: ("https://reddit.com" + .data.permalink),
    score: .data.score,
    num_comments: .data.num_comments,
    created_utc: .data.created_utc,
    link_flair_text: .data.link_flair_text,
    is_self: .data.is_self
  }]'
```

**Time windows:**
- Default: `t=hour` (posts from the last hour)
- For first run or catch-up: `t=day`
- For weekly digest: `t=week`

**Rate limiting:** Wait 2 seconds between requests. Public API allows ~10 req/min.

## Output Format

Deduplicate results by `id` across all keyword searches, then return a single JSON array:

```json
[
  {
    "id": "1abc123",
    "subreddit": "Silverbugs",
    "title": "Best app for tracking silver stack value?",
    "author": "stacker42",
    "selftext": "Looking for recommendations on apps that track...",
    "url": "https://reddit.com/r/Silverbugs/comments/1abc123/...",
    "score": 15,
    "num_comments": 23,
    "created_utc": 1740000000,
    "link_flair_text": "Discussion",
    "is_self": true,
    "matched_keywords": ["app", "tracking", "silver"]
  }
]
```

## After Scanning

Once results are collected:

1. **Report**: Summarize findings — how many relevant posts, which subreddits, trending topics
2. **Classify**: Tag each post as one of:
   - `competitor_mention` — mentions a competing app/tool by name
   - `pain_point` — user frustrated with existing tools or workflow
   - `recommendation_request` — asking for app/tool suggestions (high-value engagement opportunity)
   - `spreadsheet_user` — using spreadsheets, potential conversion target
   - `general_discussion` — general PM tracking discussion
3. **Suggest actions**: For high-value posts (recommendation requests, pain points), draft a helpful reply mentioning StackerScan's relevant features — but always present the draft for human approval before posting
4. **Log**: Append scan results summary to the daily memory file (`memory/YYYY-MM-DD.md`)

## Scheduling

This skill should run automatically every hour via the agent's heartbeat or cron. When triggered:

1. Run the scan across all subreddits
2. Deduplicate against previously seen post IDs (check today's memory file)
3. Only surface new posts
4. Report new findings to the operator

## Manual Invocation

The operator can also trigger manually:

- "Scan Reddit now" — run immediately with `t=hour`
- "Scan Reddit for the past day" — run with `t=day`
- "Scan Reddit for the past week" — run with `t=week`
- "Check r/Silverbugs for app mentions" — single subreddit scan
