# AGENTS.md

Guidance for AI agents working in this repository.

## What this repository is

Documentation and client configs for the **hosted** APITube News MCP server at
`https://mcp.apitube.io/`. The server itself is not here — it runs as a service, and this repository
is what a user or an agent reads to connect to it.

```
README.md                  what the server is, how to connect, what the tools accept
llms-install.md            install instructions written for an agent doing the setup
server.json                the entry published to registry.modelcontextprotocol.io
configs/                   ready config files, one per MCP client
assets/banner-light.svg    1280×320 header rendered at the top of the README
assets/logo-400.png        400×400 icon for directories that ask for one
```

There is no build step, no dependencies and no server code.

## Rules for editing

1. **Verify against the running server before writing anything down.** `initialize`, `tools/list`
   and `prompts/list` answer without a key:

   ```bash
   curl -s -X POST https://mcp.apitube.io/ -H 'Content-Type: application/json' \
     -d '{"jsonrpc":"2.0","method":"tools/list","id":1}'
   ```

   Tool arguments, defaults and limits come from the live `tools/list` schema, not from memory.

2. **Do not trust the docs over the server.** Where https://docs.apitube.io and the running service
   disagree, the service wins and this repository records the real behaviour.

3. **Never put a real API key in an example.** Use `YOUR_API_KEY` in configs and read
   `APITUBE_API_KEY` from the environment in scripts.

4. **Every config here must be pasteable as-is.** Test a changed config in the client it targets
   before committing it — a key in the wrong header fails as `401 ER0201` with no other hint.

5. **`server.json` mirrors the published registry entry** (`io.apitube/news`). Changing it here does
   not publish anything — the entry is published separately with `mcp-publisher`, and a change to
   tools or transport means bumping `version` in both places.

6. **Client configs are per-client, not interchangeable.** Cline needs `type: "streamableHttp"`,
   Claude Code uses `type: "http"`, Windsurf uses `serverUrl` instead of `url`, and Claude Desktop
   cannot open HTTP servers at all — it goes through the `mcp-remote` bridge. Do not collapse them
   into one snippet.

## Facts that are easy to get wrong

- Arguments are **nested objects**, never dotted keys: `{ language: { code: "en" } }`.
- Unknown argument names are **rejected** (`-32602`) with a spelling suggestion, not ignored.
- The article body is **not** in the default response — ask for it with `fl: "title,href,body"`.
- One response carries at most **25 articles**; `per_page` is clamped to 25.
- A **title search covers at most 31 days**; a wider explicit range fails with `400 ER0110`.
  Searches without a title filter have no range limit.
- `export`, `query` and `prompt` are deliberately **not** exposed as tool arguments.
- A `GET` with `Accept: text/event-stream` answers **405** on purpose: there is no SSE stream, and
  clients are expected to fall back to `POST`.

## Source of truth

In order of authority:

1. `https://mcp.apitube.io/` — run the JSON-RPC call
2. `https://api.apitube.io` — the News API underneath
3. https://docs.apitube.io/platform/news-api/ai/mcp-server — the reference
4. https://docs.apitube.io/.well-known/mcp/server-card.json — the machine-readable card
