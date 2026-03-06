---
name: reddit-monitor-pmsforsale
description: "Find engagement opportunities on r/Pmsforsale — premium calculator threads, spreadsheet users, and valuation questions."
metadata:
  {
    "openclaw":
      {
        "emoji": "💰",
        "requires": { "bins": ["curl", "jq"] }
      }
  }
---

# Reddit Monitor — r/Pmsforsale Engagement

Scans r/Pmsforsale hourly for threads where a helpful, value-first comment about StackerScan would be welcome. The goal is organic engagement, not spam.

## What to Look For

### High-value threads (engage)

1. **Premium/melt value questions** — people asking "what's this worth?" or "is this a good deal at X over spot?"
2. **Spreadsheet/tracking mentions** — users sharing Google Sheets, Excel files, or manual tracking methods
3. **Calculator requests** — asking how to calculate premiums, melt value, or spot vs buy price
4. **Photo identification** — "what do I have here?" or "can someone identify this coin/bar?"
5. **New stacker posts** — first-time buyers asking how to track or value their purchases
6. **Inventory/collection posts** — showing off stacks, discussing organization methods

### Low-value threads (skip)

- Standard [WTS]/[WTB] listings with no discussion
- Price check threads with clear answers already
- Drama, complaints about shipping/scams
- Threads older than 6 hours

## Search Queries

```bash
# Premium and valuation threads
curl -s -H "User-Agent: openclaw-agent/1.0" \
  "https://www.reddit.com/r/Pmsforsale/search.json?q=premium+OR+melt+value+OR+over+spot+OR+worth+OR+calculator&restrict_sr=on&sort=new&t=hour&limit=25" \
  | jq '[.data.children[] | {
    id: .data.id,
    title: .data.title,
    author: .data.author,
    selftext: (.data.selftext | if length > 500 then .[0:500] + "..." else . end),
    url: ("https://reddit.com" + .data.permalink),
    score: .data.score,
    num_comments: .data.num_comments,
    created_utc: .data.created_utc,
    link_flair_text: .data.link_flair_text
  }]'

# Spreadsheet and tracking threads
curl -s -H "User-Agent: openclaw-agent/1.0" \
  "https://www.reddit.com/r/Pmsforsale/search.json?q=spreadsheet+OR+tracking+OR+inventory+OR+collection+OR+organize&restrict_sr=on&sort=new&t=hour&limit=25" \
  | jq '[.data.children[] | {
    id: .data.id,
    title: .data.title,
    author: .data.author,
    selftext: (.data.selftext | if length > 500 then .[0:500] + "..." else . end),
    url: ("https://reddit.com" + .data.permalink),
    score: .data.score,
    num_comments: .data.num_comments,
    created_utc: .data.created_utc,
    link_flair_text: .data.link_flair_text
  }]'

# Photo ID and new stacker threads
curl -s -H "User-Agent: openclaw-agent/1.0" \
  "https://www.reddit.com/r/Pmsforsale/search.json?q=identify+OR+new+stacker+OR+first+purchase+OR+beginner+OR+photo&restrict_sr=on&sort=new&t=hour&limit=25" \
  | jq '[.data.children[] | {
    id: .data.id,
    title: .data.title,
    author: .data.author,
    selftext: (.data.selftext | if length > 500 then .[0:500] + "..." else . end),
    url: ("https://reddit.com" + .data.permalink),
    score: .data.score,
    num_comments: .data.num_comments,
    created_utc: .data.created_utc,
    link_flair_text: .data.link_flair_text
  }]'
```

Also scan r/Silverbugs and r/Silver for the same patterns since valuation discussions happen there too:

```bash
for sub in Silverbugs Silver; do
  curl -s -H "User-Agent: openclaw-agent/1.0" \
    "https://www.reddit.com/r/$sub/search.json?q=spreadsheet+OR+tracking+OR+premium+calculator+OR+app+OR+inventory&restrict_sr=on&sort=new&t=hour&limit=15" \
    | jq "[.data.children[] | {
      id: .data.id,
      subreddit: .data.subreddit,
      title: .data.title,
      author: .data.author,
      selftext: (.data.selftext | if length > 500 then .[0:500] + \"...\" else . end),
      url: (\"https://reddit.com\" + .data.permalink),
      score: .data.score,
      num_comments: .data.num_comments,
      created_utc: .data.created_utc
    }]"
  sleep 2
done
```

Wait 2 seconds between requests.

## Classify Each Thread

Tag each result as:

- **premium_question** — asking about premiums, melt value, fair pricing
- **spreadsheet_user** — using or sharing a spreadsheet for tracking
- **calculator_need** — wants to calculate values, doesn't have a tool
- **photo_id** — wants help identifying coins/bars from a photo
- **new_stacker** — beginner asking how to get started tracking
- **inventory_discussion** — organizing, cataloging, or showing a collection

## Draft Reply Guidelines

**Tone:** Helpful community member first, not a marketer. You are a stacker who built a tool, not a salesperson.

**Template patterns (adapt naturally to each thread — never copy-paste):**

For spreadsheet threads:
> I used to do the same thing with a spreadsheet — tracking spot prices, premiums, updating values manually. I ended up building an app that does it automatically and even lets you snap a photo to add items. Happy to share if you're interested.

For premium/calculator threads:
> If you want to check premiums quickly, I built a free tool at stackerscan.com that pulls live spot and calculates premium/melt value. No signup needed for the price charts.

For photo ID threads:
> Nice piece! If you ever want to catalog your stack quickly, there are apps now that can identify coins/bars from photos and auto-fill the details. I use one called StackerScan.

For new stacker threads:
> Welcome! Tracking gets important fast once you start accumulating. I started with a spreadsheet but switched to an app that updates values with live spot prices — way less maintenance.

**Rules:**
1. NEVER post without human approval — always present the draft and wait for confirmation
2. NEVER comment more than once per thread
3. NEVER comment on [WTS]/[WTB] posts just to promote
4. NEVER use marketing language ("revolutionary", "best app ever", "you need this")
5. ALWAYS add genuine value to the conversation first (answer their question, share knowledge)
6. ALWAYS disclose it's your own project if asked directly
7. MAX 2 comments per day across all subreddits — quality over quantity
8. Skip threads that already have 20+ comments (too late, buried)
9. Skip threads where someone already recommended a tracking tool

## Output Format

After scanning, report:

```json
{
  "scan_time": "2026-03-02T00:00:00Z",
  "threads_found": 5,
  "engagement_opportunities": [
    {
      "id": "abc123",
      "subreddit": "Pmsforsale",
      "title": "Best way to track premiums paid?",
      "url": "https://reddit.com/r/Pmsforsale/comments/abc123/...",
      "classification": "calculator_need",
      "score": 8,
      "num_comments": 3,
      "age_hours": 1.5,
      "draft_reply": "If you want to check premiums quickly...",
      "confidence": "high",
      "reason": "User explicitly asking for premium tracking tool, only 3 comments, fresh thread"
    }
  ],
  "skipped": 12,
  "skip_reasons": {
    "standard_listing": 8,
    "too_old": 2,
    "already_answered": 2
  }
}
```

## After Scanning

1. Present engagement opportunities ranked by confidence (high > medium > low)
2. Show draft reply for each high-confidence opportunity
3. **Wait for human approval** before any action
4. If approved, post using the reddit skill's OAuth2 comment endpoint (requires REDDIT_CLIENT_ID etc in .env)
5. Log posted comments to `memory/reddit-comments.md` to avoid double-posting
6. If no OAuth2 credentials, just report the opportunities and draft replies for the operator to post manually
