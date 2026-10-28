# @pipeworx/us-race-matchups

Who is on the ballot in a US House district or Senate seat: the incumbent, every FEC-registered candidate by party with the money behind them, which of them won a primary, and the primary and general election dates. Built for the 2026 midterms (general election 3 November 2026); works for any even cycle.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1856+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `race_matchup(state, office, district?, cycle?, special?)` — one race. Returns the incumbent, candidates (party, incumbent status, total receipts, disbursements, cash on hand, the date each candidate's money runs through, `won_primary`), primary result tables with vote counts and share, and the primary, runoff and general election dates. A district with no candidates returns an error naming the districts that do have candidates; a state with no Senate race that cycle says so; a race with no primary table says so in `primary_results_note` instead of returning an empty list.
- `list_races(state, office?, cycle?)` — every race in a state with how many candidates registered and who the incumbent is. Use it to find a district number.

## Auth

Platform key (the shared data.gov key the gateway already holds for the FEC pack). Callers pass nothing.

## Data sources

- FEC, candidate and committee filings — candidates, party, incumbent status, receipts, disbursements, cash on hand. Federal public record.
- FEC, election calendar — primary, runoff and general election dates.
- Wikipedia, 2026 race pages — primary vote counts and which candidates advanced. This is a volunteer-compiled source, **not an official one**; every response labels it and lists the state election authority the page cites (for example "Iowa Secretary of State"). Treat primary numbers as a convenience and check the state authority for anything that matters.

Notes the next person would otherwise rediscover:

- The FEC money view omits registrants with no financial summary on file; those appear with `financials_on_file: false` and null money (null, not zero).
- `won_primary` is INFERRED, not an id join: the primary table carries no FEC ids, so filers are matched by surname and party, narrowed by given name when several fit. `won_primary_match` says how (`inferred_by_surname_and_party`, `no_match`, `ambiguous`); when two filers cannot be told apart `won_primary` is `null`. `null` with no match field means no primary table was found.
- `incumbent` is the sitting officeholder as the FEC flags them, not necessarily someone on the ballot. Read `seeking_reelection` and `incumbent_note`: when the FEC classifies every filer as "Open seat" the incumbent is not a candidate.
- One person can hold several FEC candidate ids. `race_matchup` folds them into one candidate (main row = registration with the latest filing; the rest under `other_fec_registrations`, money not summed); `list_races` counts people, not ids.
- FEC lists some candidates' coverage dates in the future; those are shown as null and excluded from `data_as_of`.
- Large states split their Wikipedia coverage across sub-pages by district range (California); the pack follows the link.
- Ohio and Florida have both a regular and a special Senate election in 2026. FEC groups them under the state; `special: true` reads the special election's primary page.
- No home addresses or other individual contact details are served.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "us-race-matchups": {
      "url": "https://gateway.pipeworx.io/us-race-matchups/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/us-race-matchups/mcp` returns the tools in the table
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
curl -X POST https://gateway.pipeworx.io/v1/tools/race_matchup \
  -H 'Content-Type: application/json' \
  -d '{"state":"IA","office":"house","district":1}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/race_matchup`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "us-race-matchups": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-us-race-matchups"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-us-race-matchups
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Us Race Matchups data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
