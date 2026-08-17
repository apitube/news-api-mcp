<div align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/logo-dark.svg">
    <img src="assets/logo.svg" alt="APITube" width="320">
  </picture>

<h1>APITube News MCP</h1>

<p><strong>Real-time and archived news search for AI agents, over the Model Context Protocol.</strong></p>
<p>Hosted at <code>https://mcp.apitube.io/</code> — no package to install, no local process to keep alive.</p>

<p>
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue?style=flat-square" alt="License: MIT"></a>
  <img src="https://img.shields.io/badge/transport-Streamable%20HTTP-0254EC?style=flat-square" alt="Transport: Streamable HTTP">
  <img src="https://img.shields.io/badge/MCP%20protocol-2025--11--25-0254EC?style=flat-square" alt="MCP protocol 2025-11-25">
  <img src="https://img.shields.io/badge/tools-2-success?style=flat-square" alt="2 tools">
  <img src="https://img.shields.io/badge/sources-500%2C000%2B-success?style=flat-square" alt="500,000+ sources">
  <img src="https://img.shields.io/badge/languages-59-success?style=flat-square" alt="59 languages">
</p>

<p>
  <a href="#quick-start">Quick Start</a> •
  <a href="#tools">Tools</a> •
  <a href="#filters">Filters</a> •
  <a href="#prompts">Prompts</a> •
  <a href="#pricing">Pricing</a> •
  <a href="#troubleshooting">Troubleshooting</a> •
  <a href="#documentation">Docs</a> •
  <a href="#support">Support</a>
</p>

<p><strong>500,000+ sources · 177 countries · 59 languages.</strong> Sentiment and entities on every article.</p>

</div>

---

## Overview

The APITube MCP server gives an assistant live access to the world's news as structured data, not
scraped HTML. It exposes **2 tools**:

- **`search_news`** — most of the News API filter set in one call: keywords, language, country, source
  domain and quality rank, sentiment range, named entities, media, date ranges, sorting, faceting
  and highlighting.
- **`suggest`** — resolves a name like "Tesla" into the entity, category, topic and industry IDs the
  precise filters need.

Every article comes back enriched by the pipeline behind it: sentiment scores, extracted entities
(people, organizations, locations, brands, events), IPTC categories, topics and industries.

```
MCP client  →  mcp.apitube.io  →  api.apitube.io
               (this server)      (News API)

JSON-RPC over HTTP, Authorization: Bearer <API_KEY>
```

| Property | Value |
|----------|-------|
| **Endpoint** | `https://mcp.apitube.io/` |
| **Transport** | Streamable HTTP (`POST /`), JSON-RPC 2.0 |
| **Protocol** | `2025-11-25`, negotiated down to your client's version (`2024-11-05` works) |
| **Server** | `APITube News MCP-Server` `1.0.0` |
| **Auth** | `Authorization: Bearer <API_KEY>`, or `X-API-Key: <API_KEY>` |
| **Registry** | `io.apitube/news` ([`server.json`](server.json)) |

## Quick Start

1. Get an API key at [apitube.io](https://apitube.io).
2. Add the server to your client with the block below — each one is also a ready file in
   [`configs/`](configs).
3. Restart the client and ask it something like *"find positive breaking news about Tesla in English
   from the last week"*.

<details>
<summary><b>Claude Code</b></summary>

```bash
claude mcp add --transport http apitube-news https://mcp.apitube.io/ \
  --header "Authorization: Bearer YOUR_API_KEY"
```

Check it with `/mcp`. To commit the server to a project instead, put
[`configs/claude-code.mcp.json`](configs/claude-code.mcp.json) at the repo root as `.mcp.json`.

</details>

<details>
<summary><b>Cline</b></summary>

**MCP Servers → Configure**, or `~/.cline/mcp.json` for the CLI:

```json
{
  "mcpServers": {
    "apitube-news": {
      "type": "streamableHttp",
      "url": "https://mcp.apitube.io/",
      "headers": { "Authorization": "Bearer YOUR_API_KEY" },
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

`type` must be set explicitly — without it Cline falls back to the legacy SSE transport, which this
server does not serve. Both tools are read-only, so `autoApprove: ["search_news", "suggest"]` is
safe if you would rather not confirm every call.

</details>

<details>
<summary><b>Cursor</b></summary>

`~/.cursor/mcp.json` (global) or `.cursor/mcp.json` (per project):

```json
{
  "mcpServers": {
    "apitube-news": {
      "url": "https://mcp.apitube.io/",
      "headers": { "Authorization": "Bearer YOUR_API_KEY" }
    }
  }
}
```

Settings → MCP should list `apitube-news` as connected.

</details>

<details>
<summary><b>Claude Desktop</b></summary>

Claude Desktop only launches local processes, so bridge the hosted server with
[`mcp-remote`](https://www.npmjs.com/package/mcp-remote). Edit `claude_desktop_config.json`
(`~/Library/Application Support/Claude/` on macOS, `%APPDATA%\Claude\` on Windows):

```json
{
  "mcpServers": {
    "apitube-news": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://mcp.apitube.io/",
        "--header",
        "Authorization: Bearer YOUR_API_KEY"
      ]
    }
  }
}
```

Restart the app; the tools appear under the slider icon.

</details>

<details>
<summary><b>VS Code (GitHub Copilot)</b></summary>

`.vscode/mcp.json`, with the key prompted instead of stored in plain text:

```json
{
  "inputs": [
    { "type": "promptString", "id": "apitube-key", "description": "APITube API Key", "password": true }
  ],
  "servers": {
    "apitube-news": {
      "type": "http",
      "url": "https://mcp.apitube.io/",
      "headers": { "Authorization": "Bearer ${input:apitube-key}" }
    }
  }
}
```

Open Copilot Chat in **Agent** mode and enable the `apitube-news` tools.

</details>

<details>
<summary><b>Windsurf</b></summary>

`~/.codeium/windsurf/mcp_config.json` — note `serverUrl`, not `url`:

```json
{
  "mcpServers": {
    "apitube-news": {
      "serverUrl": "https://mcp.apitube.io/",
      "headers": { "Authorization": "Bearer YOUR_API_KEY" }
    }
  }
}
```

Windsurf Settings → Cascade → MCP Servers → refresh.

</details>

<details>
<summary><b>Any other client, or plain curl</b></summary>

Anything that speaks Streamable HTTP takes the URL directly; clients limited to stdio go through
`mcp-remote`, as in the Claude Desktop block. The handshake needs no key:

```bash
curl -s -X POST https://mcp.apitube.io/ \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"curl","version":"1.0.0"}},"id":1}'
```

```json
{
  "jsonrpc": "2.0",
  "result": {
    "protocolVersion": "2024-11-05",
    "serverInfo": { "name": "APITube News MCP-Server", "version": "1.0.0" },
    "capabilities": { "tools": { "listChanged": true }, "prompts": { "listChanged": true } }
  }
}
```

A real search adds the key:

```bash
curl -s -X POST https://mcp.apitube.io/ \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{"jsonrpc":"2.0","method":"tools/call","params":{"name":"search_news","arguments":{"title":"Bitcoin","language":{"code":"en"},"per_page":5}},"id":1}'
```

</details>

## Tools

| Tool | Title | Kind | What it does |
|------|-------|------|--------------|
| `search_news` | News Search | read-only | Searches articles across the News API filter set |
| `suggest` | Resolve Taxonomy IDs | read-only | Turns a name or prefix into entity / category / topic / industry IDs |

### `search_news`

Arguments are **nested objects**, never dotted strings:

```jsonc
{ "language": { "code": "en" } }      // ✅
{ "language.code": "en" }             // ❌ rejected
```

Three things worth knowing before the first call:

- **The article body is not returned by default.** The default field list is
  `id,title,href,published_at,description,source.domain`. Ask for the text explicitly with
  `fl: "title,href,body"`.
- **One response carries at most 25 articles.** `per_page` defaults to 10 and is clamped to 25; walk
  further with `page`. If a result set still has to be cut — 25 articles of full text can be large — the
  response gains an `_mcp_truncated` field saying how many were omitted.
- **A title search spans at most 31 days.** With no dates it covers the last 31 days; a wider explicit
  range fails with `400 ER0110`. Split longer periods into month-sized windows. Searches without a
  title filter have no range limit.

Misspelled arguments are **rejected** with JSON-RPC `-32602` and a suggestion, instead of being
silently ignored:

```
Unknown parameter 'langauge.code'. Did you mean 'language.code'?
```

`export`, `query` and `prompt` are deliberately not exposed.

### `suggest`

The precise filters take IDs you cannot guess, so resolve them first:

```jsonc
suggest({ type: "entities", prefix: "Tesla" })
// → [{ id: 474, name: "Tesla Robotaxi", type: "brand", … }, …]

search_news({ entity: { id: "474" }, language: { code: "en" } })
```

`type` is one of `entities`, `categories`, `topics`, `industries`; `prefix` is a name or its
beginning. Both are required. Matching is by prefix, so read the names before filtering on the
first hit.

## Filters

Everything below belongs to `search_news`. Content, taxonomy, language, author and source filters
have an `ignore.*` twin for exclusion (`ignore.title`, `ignore.entity.id`, `ignore.source.domain`, …);
sentiment, media and time filters do not. Multi-value filters take up to 3 comma-separated values.
`has_*` and `is_*` take `0` or `1`, not `true`/`false`.

<details open>
<summary><b>Content and taxonomy</b></summary>

| Argument | Example |
|----------|---------|
| `title` | `"Bitcoin"` — up to 3 comma-separated keywords, quotes for an exact phrase |
| `category.id` | `"medtop:04000000"` — IPTC taxonomy |
| `topic.id` | `"industry.crypto_news"` — slug, from `suggest` |
| `industry.id` | `"411"` — numeric, from `suggest` |
| `entity.id` | `"474"` — from `suggest` |
| `person.name` · `organization.name` · `location.name` | `"Elon Musk"` · `"Tesla,Apple"` · `"Tokyo"` |
| `brand.name` · `event.name` · `disaster.name` · `disease.name` | `"Nike"` · `"Olympics"` · `"Earthquake"` · `"COVID-19"` |
| `author.id` · `author.name` · `has_author` | `"123"` · `"Jane Smith"` · `1` |
| `language.code` | `"en,de,fr"` |

</details>

<details>
<summary><b>Sentiment</b></summary>

| Argument | Example |
|----------|---------|
| `sentiment.overall.polarity` | `"positive"` \| `"negative"` \| `"neutral"` |
| `sentiment.overall.score.{min,max}` | `-1.0` … `1.0` |
| `sentiment.title.score` · `sentiment.body.score` | same range, headline or body only |
| `sentiment.mixed` · `sentiment.consistent` | `1` — title and body disagree / agree |

</details>

<details>
<summary><b>Sources and quality</b></summary>

| Argument | Example |
|----------|---------|
| `source.domain` · `source.id` | `"cnn.com,bbc.com"` · `"314"` |
| `source.country.code` | `"us,uk,de"` |
| `source.bias` | `"left"` \| `"center"` \| `"right"` |
| `source.rank.opr.{min,max}` | OpenPageRank, 0–7 |
| `is_premium_source` · `is_verified_source` | OPR ≥ 6 · OPR ≥ 5 |
| `is_duplicate` · `is_paywall` | `0` to exclude |

</details>

<details>
<summary><b>Media, shape and time</b></summary>

| Argument | Example |
|----------|---------|
| `has_image` · `has_video` · `has_hq_images` · `is_media_rich` | `1` |
| `media.images.count.{min,max}` · `media.images.{width,height}` · `media.videos.count` | `{ "min": 2 }` |
| `is_breaking` · `is_long_read` · `is_short_read` | `1` — read time ≥ 5 min / < 3 min |
| `read_time.{min,max}` | minutes |
| `published_at.{start,end}` | `"2026-01-01"` … `"2026-01-31"`, ISO 8601 |

</details>

<details>
<summary><b>Output: sorting, paging, faceting, highlighting</b></summary>

| Argument | Example |
|----------|---------|
| `sort.by` | `published_at`, `relevance`, `engagement`, `quality`, `controversy`, `trust`, `source.rank.opr`, `sentiment.*.score`, `media.*`, `read_time`, … |
| `sort.order` | `"asc"` \| `"desc"` |
| `page` · `per_page` | `1` · `10` (max 25) |
| `fl` | `"id,title,source.name,sentiment.overall.score"` — dot notation for nested fields |
| `facet` | `true`, or `{ "field": "source.id,language.id", "limit": 20, "mincount": 5 }` |
| `facet.range` | `{ "field": "published_at", "start": "2026-01-01", "end": "2026-12-31", "gap": "1MONTH" }` |
| `hl` | `true`, or `{ "fl": "title,body", "fragsize": 300, "tag": { "pre": "<mark>", "post": "</mark>" } }` |

</details>

## Prompts

Slash commands in clients that support MCP prompts:

| Prompt | Arguments | What it does |
|--------|-----------|--------------|
| `monitor_company` | `company` (required), `days` | Recent coverage and sentiment for one company |
| `topic_sentiment` | `topic` (required), `language` | Sentiment breakdown of coverage on a topic |
| `breaking_news` | `subject`, `country` | Latest breaking stories, optionally narrowed |
| `compare_coverage` | `subject_a`, `subject_b` (both required) | Volume and sentiment, two subjects side by side |

## Use cases

| You want to | Ask for | Tools |
|-------------|---------|-------|
| Watch a brand across languages | mentions of the company with sentiment, last 7 days | `suggest` → `search_news` |
| Feed a trading or risk model | entity + industry filtered news with sentiment scores | `suggest` → `search_news` |
| Ground an agent in live news | recent articles with `fl: "title,href,body"` for RAG | `search_news` |
| Track a running story | `is_breaking: 1`, sorted by `published_at` | `search_news` |
| Measure share of voice | two subjects compared by volume and sentiment | `compare_coverage` |
| Study an archive | a date range with no title filter — no 31-day limit | `search_news` |

## Pricing

The MCP server is part of the paid plans; the free tier covers the REST API only.

| Plan | Price | Requests | MCP server |
|------|-------|----------|------------|
| Free | $0 | 100/day | — |
| Starter | $29/mo | 10,000/mo | ✅ |
| Basic | $99/mo | 50,000/mo | ✅ |
| Professional | $199/mo | 150,000/mo | ✅ |

Annual billing takes 20% off. Current numbers always live at [apitube.io/pricing](https://apitube.io/pricing).

Page size is capped separately: through MCP one response holds at most 25 articles on every plan,
regardless of the larger `per_page` the REST API allows.

## Troubleshooting

Auth and transport failures arrive as JSON-RPC `-32000` with an APITube code in the message and the
matching HTTP status.

| Code | HTTP | Meaning | Fix |
|------|------|---------|-----|
| `ER0201` | 401 | No API key reached the server | The header is missing, or the client strips custom headers — use the `mcp-remote` bridge |
| `ER0202` | 401 | Key invalid or revoked | Re-copy it from apitube.io |
| `ER0230` | 401 | Key expired | Extend the expiry in the key's settings |
| `ER0601` / `ER0602` | 403 | IP or referrer not allowed for this key | Adjust the key's restrictions |
| `ER0603` | 403 | Key not permitted to call this tool | Grant it access to `search_news` / `suggest` |
| `ER0429` | 429 | Over 120 requests/minute | Slow down — the limit is per key |
| `ER0900` | 503 | Key validation temporarily unavailable | Retry; the key is fine, do not reissue it |

<details>
<summary><b>Other symptoms</b></summary>

| Symptom | Cause |
|---------|-------|
| Client reconnects in a loop | It opened a `GET` SSE stream. Expected: the server answers `405 Allow: POST` because it has no event stream, and compliant clients fall back to `POST` |
| `-32602` with a suggested name | Misspelled argument — arguments are nested objects, never dotted keys |
| `403` from a Python script | The default `Python-urllib/3.x` user agent is rejected at the edge. Send a real `User-Agent`, or use `requests` |
| Search returns nothing for an old story | A title search only covers 31 days. Add `published_at` and walk month by month |
| Articles arrive without text | The body is excluded by default. Add `fl: "title,href,body"` |

</details>

## Documentation

| Resource | Link |
|----------|------|
| MCP server reference | https://docs.apitube.io/platform/news-api/ai/mcp-server |
| Editor setup, one-click install links | https://docs.apitube.io/platform/news-api/ai/code-editors |
| All News API parameters | https://docs.apitube.io/platform/news-api/everything |
| Authentication | https://docs.apitube.io/platform/news-api/authentication |
| Machine-readable server card | https://docs.apitube.io/.well-known/mcp/server-card.json |
| Agent skill, SDKs, migration kits | https://github.com/apitube |
| Installing this server as an agent | [`llms-install.md`](llms-install.md) |

## Support

| Channel | Where |
|---------|-------|
| Bugs and corrections | [open an issue](https://github.com/apitube/news-api-mcp/issues) |
| Account and billing | support@apitube.io |
| Everything else | https://apitube.io/contact |

Found an argument that behaves differently from what is written here? Open an issue with the request
you sent and the response you got — those corrections are the most useful thing you can file.

## License

MIT — see [LICENSE](LICENSE).
