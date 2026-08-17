# Installing the APITube News MCP server

Instructions for an AI agent installing this server on behalf of a user. The server is hosted —
there is no package to install, no repository to clone and no process to start.

## What you are configuring

| Field | Value |
|-------|-------|
| Server name | `apitube-news` |
| Transport | Streamable HTTP — the config key differs per client (`http`, `streamableHttp`) |
| URL | `https://mcp.apitube.io/` |
| Auth header | `Authorization: Bearer <APITUBE_API_KEY>` |
| Alternative header | `X-API-Key: <APITUBE_API_KEY>` |
| Tools | `search_news`, `suggest` (both read-only) |

## Step 1 — get the key from the user

Ask the user for their APITube API key. Do not invent one, and do not proceed without it: every
`tools/call` fails with `401 ER0201` when the key is missing.

If the user has no key, send them to https://apitube.io to sign up. The MCP server requires a paid
plan (Starter and up) — https://apitube.io/pricing.

Never write the key into a file the user tracks in git. In VS Code use the `promptString` input
shown below; elsewhere, put it in the client's own config file, which lives outside the project.

## Step 2 — write the config

Use the block for the client you are installing into. Replace `YOUR_API_KEY` with the user's key.

**Claude Code** — run:

```bash
claude mcp add --transport http apitube-news https://mcp.apitube.io/ \
  --header "Authorization: Bearer YOUR_API_KEY"
```

**Cline** — merge into the MCP settings JSON (MCP Servers → Configure in the panel; `~/.cline/mcp.json`
for the CLI), keeping the servers already there. `type` must be set explicitly, otherwise Cline falls
back to the legacy SSE transport, which this server does not serve:

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

**Cursor** — `~/.cursor/mcp.json` (global) or `.cursor/mcp.json` (project); no `type` key needed:

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

**Windsurf** — `~/.codeium/windsurf/mcp_config.json`, same shape but the URL key is `serverUrl`:

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

**Claude Desktop** cannot open HTTP servers itself. Bridge it with `mcp-remote`:

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

`claude_desktop_config.json` lives in `~/Library/Application Support/Claude/` on macOS and
`%APPDATA%\Claude\` on Windows.

**VS Code (GitHub Copilot)** — `.vscode/mcp.json`, key prompted rather than stored:

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

## Step 3 — verify

The handshake needs no key, so it isolates transport problems from auth problems:

```bash
curl -s -X POST https://mcp.apitube.io/ \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"install-check","version":"1.0.0"}},"id":1}'
```

A healthy server answers with `"serverInfo": {"name": "APITube News MCP-Server", "version": "1.0.0"}`.

Then check the key end to end:

```bash
curl -s -X POST https://mcp.apitube.io/ \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{"jsonrpc":"2.0","method":"tools/call","params":{"name":"search_news","arguments":{"title":"Bitcoin","per_page":2}},"id":1}'
```

Finally, restart the client (or reload its MCP servers) and confirm `search_news` and `suggest`
appear in its tool list.

## Step 4 — tell the user how to drive it

One sentence is enough: they ask in plain language ("find positive breaking news about Tesla in
English from the last week") and the assistant calls `search_news`. Mention two behaviours that
surprise people otherwise:

- Article text is not returned unless asked for — `fl: "title,href,body"`.
- One response holds at most 25 articles; `per_page` is clamped to 25 and further results come from
  `page`.

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| `401 ER0201` | No key reached the server | The header is missing from the config, or the client strips custom headers — use the `mcp-remote` bridge |
| `401 ER0202` | Key invalid or revoked | Re-copy it from apitube.io |
| `401 ER0230` | Key expired | Extend the expiry in the key's settings |
| `403 ER0603` | Key not permitted to call the tool | Grant the key access to `search_news` / `suggest` |
| `429 ER0429` | Over 120 requests/minute | Slow down; the limit is per key |
| `503 ER0900` | Key validation temporarily unavailable | Retry — the key itself is fine, do not reissue it |
| Client reconnects in a loop | It opened a `GET` SSE stream | Expected: the server answers `405` because it has no SSE stream, and compliant clients fall back to `POST` |
| `-32602` with a suggested name | Misspelled argument | Use the suggestion; arguments are nested objects (`{ language: { code: "en" } }`), never dotted keys |
