# @pipeworx/jp-mof-asset-freeze

Japan's asset-freeze target list (資産凍結等の措置の対象者), published by the
Ministry of Finance (財務省) under the Foreign Exchange and Foreign Trade Act:
about 2,900 individuals and organisations, searchable by Japanese or English
name, alias or former name. Entries come back exactly as the ministry publishes
them.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2016+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `jp_asset_freeze_search(name, limit?)` — exact or partial name match after
  Unicode folding and dropping corporate-form words (株式会社, Co., Ltd.).
  A katakana spelling that differs from the ministry's (ヴ/ブ, long-vowel marks,
  middle dots, hiragana, or katakana against the listed English name, e.g.
  ワグネル for the listed ワグナー / Wagner) comes back as `match: "variant"`
  with `variant_kind` (`kana` or `phonetic`) and `matched_form`, the listed
  spelling it matched; variants rank after exact and partial hits and are leads
  to read, never a determination.
  Returns `result` (`hit` / `no_hit`), `total_matches`, the matching entries
  (names, aliases, former names, nationality, address, UN reference, notice
  date and number), `list_size` and `data_as_of`.

## Auth

Keyless.

## Data sources

- Japan Ministry of Finance (財務省), asset-freeze target list under the
  Foreign Exchange and Foreign Trade Act. Read from the publisher on request;
  the list in use is cached in the serving process for an hour. `data_as_of`
  is the date of the list edition the ministry published.

A hit is a lead to read against the ministry's own notice, not a determination
that the party is the listed one. No hit means no entry matched this spelling,
variants included, on `data_as_of`; the list may spell a foreign name differently
still, so it is not a clearance and says nothing about other countries' lists (use
`sanctions_screen` for US lists). The list carries named individuals as the
ministry publishes them; nothing is added to an entry.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "jp-mof-asset-freeze": {
      "url": "https://gateway.pipeworx.io/jp-mof-asset-freeze/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/jp-mof-asset-freeze/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 2016+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/jp_asset_freeze_search \
  -H 'Content-Type: application/json' \
  -d '{"name":"Sberbank"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/jp_asset_freeze_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-jp-mof-asset-freeze` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/jp-mof-asset-freeze/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "jp-mof-asset-freeze": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-jp-mof-asset-freeze"]
    }
  }
}
```

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Jp Mof Asset Freeze data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
