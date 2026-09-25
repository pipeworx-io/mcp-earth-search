# @pipeworx/earth-search

Sentinel and Landsat imagery catalogue from Element 84's Earth Search over the
AWS Open Data registry — search Sentinel-1/2, Landsat Collection 2, NAIP and
Copernicus DEM by area and date and get item ids, footprints, cloud cover and
cloud-optimised GeoTIFF URLs.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1679+ live data sources.

## Tools

- `earth_search_collections(contains?, prefix?, limit?)` — the missions indexed,
  with licence, extent and date range.
- `earth_search_collection(collection_id)` — one collection in full, including
  the asset keys (bands, thumbnails) its items carry.
- `earth_search_search(collections?, bbox?, datetime?, ids?, limit?, max_assets?)`
  — which scenes cover an area on given dates. `collections` is **optional**
  here, unlike the sibling catalogues.

## Auth

Keyless, metadata *and* assets. Hrefs resolve on AWS Open Data with no token and
no account — this is the catalogue to send a caller to when they need bytes
rather than metadata. Some buckets are requester-pays when read over `s3://`
rather than `https://`.

## Data sources

- <https://earth-search.aws.element84.com/v1> — STAC API 1.0.

Shared client: `shared/src/stac.ts`, also used by `planetary-computer`,
`copernicus-dataspace` and `meteoswiss`.

## Traps

- **This is the one catalogue in the set that allows a catalogue-wide search.**
  Planetary Computer (422) and the Copernicus Data Space (400) both refuse a
  search with no `collections`. Useful for "is there ANY imagery here" — but the
  result then spans missions with very different resolutions, so read
  `collection` on each item before comparing them.
- Collection ids are **not portable**. Sentinel-2 L2A is `sentinel-2-c1-l2a`
  here (Collection 1 reprocessing) and `sentinel-2-pre-c1-l2a` for the older
  baseline, while the same mission is `sentinel-2-l2a` on Planetary Computer and
  on the Copernicus Data Space. The wrong id returns an empty result, not an
  error.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "earth-search": {
      "url": "https://gateway.pipeworx.io/earth-search/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/earth-search/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1679+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/earth_search_collections \
  -H 'Content-Type: application/json' \
  -d '{"contains":"sentinel-2","limit":5}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/earth_search_collections`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "earth-search": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-earth-search"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-earth-search
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Earth Search data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
