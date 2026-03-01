---
name: reddit
description: "Browse and post to Reddit: trending posts, subreddit search, read threads, submit posts and comments."
metadata:
  {
    "openclaw":
      {
        "emoji": "📡",
        "requires": { "bins": ["curl", "jq"] }
      }
  }
---

# Reddit

Read and post to Reddit using the API. Reading works without auth via public JSON endpoints. Posting requires OAuth2 credentials.

## Reading (no auth required)

All read endpoints append `.json` to normal Reddit URLs. Always include a `User-Agent` header.

### Trending / hot posts in a subreddit

```bash
curl -s -H "User-Agent: openclaw-agent/1.0" "https://www.reddit.com/r/SUBREDDIT/hot.json?limit=10" | jq '.data.children[] | {title: .data.title, score: .data.score, url: .data.url, comments: .data.num_comments, author: .data.author}'
```

### New posts

```bash
curl -s -H "User-Agent: openclaw-agent/1.0" "https://www.reddit.com/r/SUBREDDIT/new.json?limit=10" | jq '.data.children[] | {title: .data.title, score: .data.score, url: .data.url, author: .data.author}'
```

### Top posts (time filter: hour, day, week, month, year, all)

```bash
curl -s -H "User-Agent: openclaw-agent/1.0" "https://www.reddit.com/r/SUBREDDIT/top.json?t=week&limit=10" | jq '.data.children[] | {title: .data.title, score: .data.score, url: .data.url, author: .data.author}'
```

### Search Reddit

```bash
curl -s -H "User-Agent: openclaw-agent/1.0" "https://www.reddit.com/search.json?q=QUERY&sort=relevance&limit=10" | jq '.data.children[] | {title: .data.title, subreddit: .data.subreddit, score: .data.score, url: .data.url}'
```

### Search within a subreddit

```bash
curl -s -H "User-Agent: openclaw-agent/1.0" "https://www.reddit.com/r/SUBREDDIT/search.json?q=QUERY&restrict_sr=on&limit=10" | jq '.data.children[] | {title: .data.title, score: .data.score, url: .data.url}'
```

### Read a post and its comments

```bash
curl -s -H "User-Agent: openclaw-agent/1.0" "https://www.reddit.com/r/SUBREDDIT/comments/POST_ID.json" | jq '.[0].data.children[0].data | {title, selftext, score, author, num_comments}'
```

For comments on that post:

```bash
curl -s -H "User-Agent: openclaw-agent/1.0" "https://www.reddit.com/r/SUBREDDIT/comments/POST_ID.json" | jq '.[1].data.children[] | {author: .data.author, body: .data.body, score: .data.score}'
```

### User profile

```bash
curl -s -H "User-Agent: openclaw-agent/1.0" "https://www.reddit.com/user/USERNAME/submitted.json?limit=10" | jq '.data.children[] | {title: .data.title, subreddit: .data.subreddit, score: .data.score}'
```

### Popular across all of Reddit

```bash
curl -s -H "User-Agent: openclaw-agent/1.0" "https://www.reddit.com/r/popular/hot.json?limit=10" | jq '.data.children[] | {title: .data.title, subreddit: .data.subreddit, score: .data.score, url: .data.url}'
```

## Posting (requires OAuth2)

Posting requires env vars: `REDDIT_CLIENT_ID`, `REDDIT_CLIENT_SECRET`, `REDDIT_USERNAME`, `REDDIT_PASSWORD`.

### Get an access token

```bash
TOKEN=$(curl -s -X POST "https://www.reddit.com/api/v1/access_token" \
  -u "$REDDIT_CLIENT_ID:$REDDIT_CLIENT_SECRET" \
  -H "User-Agent: openclaw-agent/1.0" \
  -d "grant_type=password&username=$REDDIT_USERNAME&password=$REDDIT_PASSWORD" \
  | jq -r '.access_token')
```

The token is valid for ~1 hour. Reuse it across multiple calls in the same session.

### Submit a text post

```bash
curl -s -X POST "https://oauth.reddit.com/api/submit" \
  -H "Authorization: Bearer $TOKEN" \
  -H "User-Agent: openclaw-agent/1.0" \
  -d "sr=SUBREDDIT&kind=self&title=POST_TITLE&text=POST_BODY"
```

### Submit a link post

```bash
curl -s -X POST "https://oauth.reddit.com/api/submit" \
  -H "Authorization: Bearer $TOKEN" \
  -H "User-Agent: openclaw-agent/1.0" \
  -d "sr=SUBREDDIT&kind=link&title=POST_TITLE&url=LINK_URL"
```

### Comment on a post

The `thing_id` is the post's fullname (e.g. `t3_abc123`). Get it from the post JSON: `.data.children[0].data.name`.

```bash
curl -s -X POST "https://oauth.reddit.com/api/comment" \
  -H "Authorization: Bearer $TOKEN" \
  -H "User-Agent: openclaw-agent/1.0" \
  -d "thing_id=t3_POST_ID&text=COMMENT_BODY"
```

### Reply to a comment

Use the comment's fullname (`t1_xyz789`) as `thing_id`:

```bash
curl -s -X POST "https://oauth.reddit.com/api/comment" \
  -H "Authorization: Bearer $TOKEN" \
  -H "User-Agent: openclaw-agent/1.0" \
  -d "thing_id=t1_COMMENT_ID&text=REPLY_BODY"
```

### Vote on a post or comment

Direction: `1` (upvote), `0` (unvote), `-1` (downvote).

```bash
curl -s -X POST "https://oauth.reddit.com/api/vote" \
  -H "Authorization: Bearer $TOKEN" \
  -H "User-Agent: openclaw-agent/1.0" \
  -d "id=THING_ID&dir=1"
```

## Rate limits

- Public JSON endpoints: ~10 requests/minute without auth
- OAuth2 endpoints: ~60 requests/minute
- Always respect `X-Ratelimit-Remaining` and `X-Ratelimit-Reset` headers
- Add 1-2 second delays between consecutive requests

## Tips

- Replace SUBREDDIT, QUERY, POST_ID, USERNAME with actual values
- URL-encode query strings and post bodies when they contain special characters
- For long text bodies, use `--data-urlencode` instead of `-d`
- The `after` parameter in JSON responses enables pagination: append `&after=FULLNAME` to get the next page
