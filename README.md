# mcp-opentripmap

OpenTripMap MCP — wraps the OpenTripMap Places API (dev.opentripmap.org)

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1576+ live data sources.

## Tools

| Tool | Description |
|------|-------------|
| `opentripmap_places_radius` | Find points of interest (tourist attractions, museums, restaurants, historic sites, etc.) within a radius of a lat/lon point. Returns matching POIs with xid, name, kind categories, distance from the center, and coordinates. Example: opentripmap_places_radius({ lat: 48.8566, lon: 2.3522, radius: 1000, kinds: "museums", limit: 20, _apiKey: "your-key" }) |
| `opentripmap_place_detail` | Get full details for a single place by its OpenTripMap xid (obtained from opentripmap_places_radius or opentripmap_autosuggest). Returns name, address, categories, Wikidata/Wikipedia links, description extract, image, and website. Example: opentripmap_place_detail({ xid: "N123456", _apiKey: "your-key" }) |
| `opentripmap_geoname` | Resolve a city / place name to geographic coordinates and metadata (lat, lon, population, country, timezone). Use this to turn a place name into a lat/lon you can feed to opentripmap_places_radius. Example: opentripmap_geoname({ name: "Paris", country: "FR", _apiKey: "your-key" }) |
| `opentripmap_autosuggest` | Typeahead / partial-name search for POIs near a lat/lon point. Matches place names by prefix within a radius — good for "find places starting with X near here". Returns matching POIs with xid, name, kinds, distance, and coordinates. Example: opentripmap_autosuggest({ name: "louv", lat: 48.8566, lon: 2.3522, radius: 5000, _apiKey: "your-key" }) |

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "opentripmap": {
      "url": "https://gateway.pipeworx.io/opentripmap/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/opentripmap/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1576+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

This pack takes your own API key (`_apiKey`) — we don't front one for it, so there's no curl here that would run without it. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/opentripmap_places_radius`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "opentripmap": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-opentripmap"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-opentripmap
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Opentripmap data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
