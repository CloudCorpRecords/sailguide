# sailguide

Plan a crewed charter in the British Virgin Islands around one yacht, without leaving your tool list. sailguide is a local forwarding client for the hosted Dream On charter concierge service: it answers `tools/list` locally from the captured tool definitions and forwards every call to the concierge, returning the answers exactly as the service sends them back.

The five tools are the concierge's information core. You can read the yacht's published fact sheet, check whether a set of dates is open, search a whole season for open windows, get a price estimate from the published rate rules, and — when Dream On itself does not fit — follow a pre-filled link to its sister concierge for other hand-picked BVI yachts. Nothing is booked, held or sent from here: this package is a pipe to the concierge's published information, not a booking agent.

This is a labelled subset: 5 of the 7 captured tools are built. The two omitted tools are not registered, listed or callable anywhere in this package.

## The tools

- **`get_dream_on_facts`** — Return Dream On's published fact sheet: the yacht, cabins, toys, amenities, crew, what is included, rate rules, booking process, a sample itinerary and the FAQ. No parameters are required; an optional `section` narrows the answer.
- **`check_availability`** — Check whether specific dates are open. `start` is required; give `end` or `nights`. When the dates are not open, the service suggests open windows nearby.
- **`find_open_windows`** — List open windows between two dates for charters of at least `min_nights`. `from` and `to` are required.
- **`get_quote`** — Estimate the all-inclusive charter fee from the published rate rules. `guests`, `start` and `nights` are required. When the service cannot price a request it says so rather than inventing a figure.
- **`suggest_other_yachts`** — Return a link to Islands & Sails, Dream On's sister concierge, pre-filled with the guest's dates and party size. No parameters are required. The link submits nothing.

Every tool also accepts an optional `requester` object describing who the request is for; sailguide forwards it untouched, exactly as the caller supplied it.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/sailguide mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "sailguide": {
      "command": "npx",
      "args": ["-y", "@reneturcios/sailguide", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/sailguide
docker run --rm ghcr.io/cloudcorprecords/sailguide get_dream_on_facts '{}'
```

## CLI

One binary serves both the command line and the MCP server. Each tool is a subcommand taking a single JSON string with that tool's arguments:

```bash
sailguide get_dream_on_facts '{}'
sailguide check_availability '{"start":"2027-05-22","nights":7}'
sailguide get_quote '{"guests":6,"start":"2027-05-22","nights":7}'
sailguide --version
```

Exit codes: `0` when the forwarded call succeeds, `1` when the upstream result is an error, and `2` for usage problems such as an unknown subcommand or a malformed JSON argument.

## Configuration

- `DREAMONSAIL_MCP_URL` — Overrides the upstream endpoint. The default is the pinned hosted endpoint, `https://dreamonsail.com/mcp`. (The override variable is a reconstruction added by this rebuild.)

The upstream registry record declares no headers, so no authentication header is ever sent; the tool surface was captured keyless.

Upstream failures — a JSON-RPC error from the service, a non-2xx response, an unreadable body, or a request that fails outright — are reported to the caller as an error result whose text starts with `sailguide error:`.

## How forwarding works

The first forwarded call opens a session: an `initialize` request (protocol version `2025-03-26`), followed by a `notifications/initialized` notification, then the `tools/call` itself. If the service answers the initialize request with an `Mcp-Session-Id` header, that value is carried on every later request, and the open session is reused for the rest of the process. Replies are accepted either as plain JSON or as server-sent events. Successful results pass through verbatim, including any `isError` flag and any extra fields the service includes.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The test suite checks the served tool list against the captured surface (all 5 tools, deep-equal, in order, with the empty `required` arrays of the two parameterless tools served verbatim), asserts that the two omitted tools are absent at registry, MCP and CLI level, exercises the session handshake and its reuse, verifies that no `Authorization` header is ever sent, and verifies that arguments go upstream untouched and results come back untouched.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
