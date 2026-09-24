# @pipeworx/511-sf-bay

Live departures, vehicle positions and service alerts for every transit
operator in the San Francisco Bay Area, from the regional 511.org open-data
API run by the Metropolitan Transportation Commission.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1679+ live data sources.

**Requires an API key** — free, self-service and issued instantly at
<https://511.org/open-data/token>.

- Platform key: `PLATFORM_511_KEY`
- Caller-supplied: `_apiKey` argument on any tool

## Tools

| Tool | What it answers |
|---|---|
| `bay511_operators` | Which agencies publish here, and their agency codes |
| `bay511_stop_predictions` | Live predicted departures from one stop |
| `bay511_vehicles` | Where an operator's vehicles are right now |
| `bay511_alerts` | Current disruptions, with cause and effect |

## One pack for ~30 agencies, on purpose

511 is the Bay Area's single regional aggregator — BART, Muni, Caltrain, AC
Transit, VTA, SamTrans, Golden Gate and two dozen more all publish through it
behind an `agency` code. Splitting it per operator would be thirty packs over
one credential and one upstream. Call `bay511_operators` first to get the code.

## Traps

**The JSON is served with a UTF-8 BOM** and `JSON.parse` throws on it. Stripped
in `readJson()`; do not "simplify" that away.

**Two response families on one host and key.** Departures and vehicles are SIRI
JSON. Alerts are GTFS-Realtime **protobuf** and go through the shared decoder
in `@pipeworx/shared` (`fetchGtfsRt`). Same key, different content type.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "511-sf-bay": {
      "url": "https://gateway.pipeworx.io/511-sf-bay/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/511-sf-bay/mcp` returns the tools in the table
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
curl -X POST https://gateway.pipeworx.io/v1/tools/bay511_operators \
  -H 'Content-Type: application/json' \
  -d '{}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/bay511_operators`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "511-sf-bay": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-511-sf-bay"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-511-sf-bay
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about 511 Sf Bay data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
