# @pipeworx/police-shootings

U.S. fatal police shootings since 2015 — aggregate counts, individual
incident records (including the name of the person killed, as published),
and per-agency breakdowns, proxied live from The Washington Post's Fatal
Force database.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1856+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

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

Both URLs reach the same gateway and the same 1856+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/police_shootings_summary \
  -H 'Content-Type: application/json' \
  -d '{"group_by":"year"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/police_shootings_summary`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-police-shootings` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/police-shootings/mcp` endpoint shown above. It needs network access;
there is no offline mode.

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
ask_pipeworx({ question: "your question about Police Shootings data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
