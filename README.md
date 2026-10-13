# @pipeworx/police-shootings

U.S. fatal police shootings since 2015 — aggregate counts, individual
incident records (including the name of the person killed, as published),
and per-agency breakdowns, proxied live from The Washington Post's Fatal
Force database.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1784+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `police_shootings_summary(group_by?, state?, year?)` — aggregate counts
  (by state, year, race, armed_with, threat_type, gender, mental_illness, or
  body_camera). Never returns names.
- `police_shootings(state?, city?, year?, armed_with?, race?, limit?, offset?)`
  — individual incident records, including the name of the person killed.
  Small by default (limit 25, max 200); a truncated response carries
  `truncated:true`, `total_matching`, and a narrowing hint.
- `agency_shootings(agency, state?)` — one law-enforcement agency's
  fatal-shooting count and breakdown by year/race/armed_with. Matches on a
  name fragment; returns a small list of candidates when more than one
  agency matches, rather than guessing.

## Auth

Keyless. The upstream is a public CSV with no registration, key, or login.

## Data sources

- The Washington Post, Fatal Force database — fatal shootings by on-duty
  police officers in the United States since Jan. 1, 2015, plus the roster
  of law-enforcement agencies involved. Published under
  Attribution-NonCommercial-ShareAlike 4.0 International (CC BY-NC-SA 4.0);
  every tool response carries `license` and `attribution` fields so the
  terms travel with the data. This pack is a LIVE PROXY — it fetches the
  Post's published files fresh on every cache miss (6h edge cache, same as
  any normal client would cache) and never commits the dataset into this
  repo (CLAUDE.md "mirror-needs-grant-proxy-does-not": a per-request proxy
  call is not redistribution; bulk-copying the dataset would be, and the
  licence forbids that). Human-facing page:
  <https://www.washingtonpost.com/graphics/investigations/police-shootings-database/>.
  Routes live only in `src/index.ts`.

## Notes

- The Post's own README states the data is "updated regularly," but the
  published file this pack reads currently ends 2024-12-31 (verified live
  2026-10-13) — there is no newer data to proxy. Every response's
  `data_as_of` / `dataset_coverage.max_date` reflects whatever the Post has
  actually published at call time, not a fixed claim.
- The Post tracks only on-duty police shootings paralleling the Michael
  Brown case — not deaths in custody, off-duty shootings, or non-shooting
  deaths. It is not comprehensive beyond that scope (the Post itself notes
  undercounting is possible before reporting catches up).
- `race` and `armed_with` can carry more than one value per incident,
  semicolon-separated in the source (e.g. `"W;B"`, `"gun;knife"`). The
  `race` and `armed_with` filters in `police_shootings`/`police_shootings_summary`
  match if the requested value is ANY one of the semicolon-separated tokens.
- `agency_ids` on an incident can list more than one agency
  (semicolon-separated) when multiple departments were involved;
  `agency_shootings` joins on that list.
- Names, ages, and other person-level fields are served exactly as the Post
  publishes them — nothing is added, inferred, or joined against any other
  people-data source.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "police-shootings": {
      "url": "https://gateway.pipeworx.io/police-shootings/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/police-shootings/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1784+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/police_shootings_summary \
  -H 'Content-Type: application/json' \
  -d '{"group_by":"year"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/police_shootings_summary`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "police-shootings": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-police-shootings"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-police-shootings
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Police Shootings data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
