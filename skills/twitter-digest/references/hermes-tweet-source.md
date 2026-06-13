# Hermes Tweet Source

Use Hermes Tweet as an optional live source for `/twitter-digest` when the user wants fresh bookmarks or better article/thread enrichment without handling browser cookies inside this skill.

Hermes Tweet is a Hermes Agent plugin for X/Twitter automation. It exposes:

- `tweet_explore` for endpoint discovery without a network call.
- `tweet_read` for catalog-listed public read endpoints.
- `tweet_action` for private reads and writes, disabled unless explicitly enabled.

## When to Use

Use this route when:

- The inbox has no export file and the user asks to fetch fresh bookmarks.
- A bookmark is an X Article or thread and the export only contains a short URL or partial text.
- The user wants richer tweet metadata before categorization.
- Hermes Agent already has the `hermes-tweet` toolset available.

Do not use this route for casual Twitter search, posting, liking, following, DMs, or profile changes. This skill is for bookmark ingestion and enrichment only.

## Setup Check

Run a quiet preflight before fetching:

```bash
hermes tools list
```

Look for the `hermes-tweet` toolset. If missing, suggest:

```bash
hermes plugins install Xquik-dev/hermes-tweet --enable
hermes tools list
```

Never ask the user to paste `XQUIK_API_KEY`, browser cookies, bearer tokens, `auth_token`, or `ct0` into chat. If a key is missing, tell the user to configure it in the Hermes runtime environment.

## Capability Discovery

Always discover endpoints before calling them:

```json
{"query":"bookmarked tweets","include_actions":true}
```

Expected endpoint for bookmark ingestion:

```text
GET /api/v1/x/bookmarks
```

Bookmarks are private account data, so this endpoint is routed through `tweet_action`, even though the HTTP method is `GET`.

Useful enrichment discovery queries:

```json
{"query":"X article content by tweet id","method":"GET"}
{"query":"full tweet thread by tweet id","method":"GET"}
{"query":"get tweet by id full text metrics media","method":"GET"}
{"query":"get multiple tweets by ids","method":"GET"}
```

Expected read endpoints:

```text
GET /api/v1/x/articles/{tweetId}
GET /api/v1/x/tweets/{id}
GET /api/v1/x/tweets/{id}/thread
GET /api/v1/x/tweets
```

## Fetch Bookmarks

Before the private read, restate the endpoint and purpose:

```text
I will fetch bookmarked tweets with GET /api/v1/x/bookmarks and save a normalized JSON export in the bookmark inbox. This reads private account data and will not post, like, follow, or send messages.
```

Only proceed after approval. Then call `tweet_action`:

```json
{
  "path":"/api/v1/x/bookmarks",
  "method":"GET",
  "query":{"limit":100},
  "reason":"Fetch user-approved bookmarked tweets for twitter-digest ingestion."
}
```

If pagination is available in the response, continue until the requested batch is complete or the endpoint reports no next page. Use conservative page sizes and stop on auth, policy, account-state, or rate-limit errors. Do not retry through another backend after a policy/auth/account-state error.

## Normalize Output

Write normalized bookmarks to:

```text
$INBOX_DIR/hermes-tweet-bookmarks-YYYYMMDD.json
```

Use this shape:

```json
[
  {
    "text":"Full tweet text or article text when available",
    "author":"Display Name",
    "author_handle":"username",
    "url":"https://x.com/username/status/123",
    "date":"2026-05-23T09:00:00Z",
    "source":"hermes-tweet",
    "tweet_id":"123",
    "metrics":{"likes":10,"reposts":2,"replies":1,"quotes":0},
    "raw_ref":"tweet_read:/api/v1/x/tweets/123"
  }
]
```

Keep raw API payloads out of the vault unless needed for a specific field. Do not write credentials, request headers, cookies, or API keys into the inbox, vault, or summary.

## Enrich Articles

For `x.com/i/article/...` or URL-only article bookmarks:

1. Extract the tweet ID from the surrounding bookmark URL when present.
2. Discover the article endpoint with `tweet_explore`.
3. Call `tweet_read`:

```json
{"path":"/api/v1/x/articles/1234567890"}
```

4. Replace URL-only text with the article title and plain text.
5. Preserve the original URL and add `source:"hermes-tweet-article"`.

If article enrichment fails, keep the bookmark and categorize from available author, URL, and surrounding text.

## Enrich Threads

For thin thread bookmarks or posts whose value depends on replies:

```json
{"path":"/api/v1/x/tweets/1234567890/thread"}
```

Collapse the thread into one digest candidate:

- Put the root tweet first.
- Append follow-up tweets in chronological order.
- Keep reply author handles when they differ from the root author.
- Preserve source URLs for every tweet that contributes a claim.

Do not inflate one thread into many vault entries unless it contains clearly separate durable ideas.

## Enrich Single Tweets

For partial exports:

```json
{"path":"/api/v1/x/tweets/1234567890"}
```

Use this to fill missing author handles, dates, metrics, media URLs, or full text. For multiple IDs, discover and use `/api/v1/x/tweets` with query parameters instead of calling single-tweet reads one by one.

## Failure Handling

| Failure | Action |
| --- | --- |
| `tweet_explore` missing | Tell the user Hermes Tweet is not installed or not enabled. Use export/script fallback. |
| `tweet_read` missing | Ask the user to configure `XQUIK_API_KEY` in Hermes runtime. Do not ask for the value. |
| `tweet_action` disabled | Explain that private bookmark reads require `HERMES_TWEET_ENABLE_ACTIONS=true` and explicit approval. |
| Auth or account-state error | Stop. Do not retry with cookies, GraphQL, or another backend. |
| Rate limit | Stop or reduce page size. Do not loop aggressively. |
| Article/thread not found | Keep the original bookmark and mark enrichment as unavailable. |

## Summary Line

After a Hermes Tweet fetch, report:

```text
OK hermes-tweet - fetched=X normalized=Y enriched=Z skipped=W
```

Then continue with the normal `/twitter-digest` categorization workflow.
