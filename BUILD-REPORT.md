# BUILD REPORT — sailguide

Clean-room rebuild of `com.dreamonsail/concierge` v1.1.0
(remote-only MCP server at `https://dreamonsail.com/mcp`), built
2026-10-28 as a forwarding client. Package: `@reneturcios/sailguide`
0.1.0, registry name `io.github.CloudCorpRecords/sailguide`.

## Surface built

Labelled subset: **5 of the 7 captured tools built**; 2 omitted
(see below).

| Tool | Required fields as served |
|------|---------------------------|
| `get_dream_on_facts` | `[]` — empty array in the capture, served verbatim |
| `check_availability` | `start` |
| `find_open_windows` | `from`, `to` |
| `get_quote` | `guests`, `start`, `nights` |
| `suggest_other_yachts` | `[]` — empty array in the capture, served verbatim |

Each served tool carries exactly the keys `name`, `title`,
`description`, `inputSchema`, exactly as captured: the capture's
`annotations` key was dropped at fixture slimming and is NOT part
of the served literals. The tool literals were generated
programmatically from
`harness/fixtures/dreamonsail-tools-original.json` by the script at
`/tmp/b45-sailguide-scratch/gen.py` (parse + re-serialise into
`src/forward.ts`) — never retyped — and a post-generation check
(the registration-integrity test suite, plus the parity harness)
confirmed the served literals deep-equal the fixture's 5 tool
objects, in order. Tools are registered via the MCP SDK low-level
Server API (`ListToolsRequestSchema` / `CallToolRequestSchema`
handlers), not McpServer+zod.

## Labelled subset — omissions

The capture enumerates 7 tools. Two are OMITTED from the fixture
and from this rebuild, and are not registered, listed or callable
anywhere (registry, MCP tools/list, or CLI — absence is asserted
in tests at all three levels):

- `request_hold` — passes a guest's full name, email, phone, dates
  and party size to the owner and to a clearing house (Paradise
  Yacht Clearing) to place a 48-hour charter hold. A booking-shaped
  SEND of a named person's contact details (the PII lead-routing /
  booking-execution family).
- `ask_owner` — passes a guest's name and email with a free-text
  question to the owner for a reply. A send tool carrying a named
  person's contact details.

The five kept tools are the concierge's information core: the
published fact sheet, availability checks, open-window search, a
published-rate quote estimate, and a sister-concierge link read
(the link is returned pre-filled and submits nothing). Every kept
tool's schema carries an OPTIONAL caller-supplied `requester`
object; it is served verbatim and merely forwarded — the rebuild
never originates a follow-up, and the two tools that actually
transmit anything to the owner are the omitted ones.

Note (fixture verbatim, not a registration): the captured
description of the kept tool `get_quote` mentions `ask_owner` in
its prose ("offer to pass the question to the owner with
ask_owner"). That sentence is part of the fixture's verbatim
description literal and is served exactly as captured under
THE FIXTURE WINS; the `ask_owner` tool itself is not registered,
listed or callable.

## Fixture conflicts

**None.** The spec and the fixture agree on all 5 kept tools'
names, titles, descriptions, required fields (including the two
empty `required` arrays on `get_dream_on_facts` and
`suggest_other_yachts`) and property shapes, and on the two
omitted tools recorded in the fixture's `_source` note.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://dreamonsail.com/mcp`.
- Env override `DREAMONSAIL_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent. Tests assert that no `Authorization` header
  is sent under any environment — including with key-like env vars
  (`DREAMONSAIL_API_KEY`, `API_KEY`, `AUTHORIZATION`) set — and that
  no `X-API-Key` header is ever sent; upstream requests carry only
  `Content-Type`, `Accept` and, when issued, `Mcp-Session-Id`.
- Error rendering `sailguide error:` text for upstream failures —
  labelled reconstruction.
- Upstream results pass through verbatim, including `isError` and
  extra fields. tools/call is forwarded as streamable-HTTP
  JSON-RPC: initialize (protocolVersion `2025-03-26`) →
  notifications/initialized → tools/call, honouring
  `Mcp-Session-Id` when issued; plain-JSON and SSE replies are
  both parsed. The forwarding core takes an injectable fetch;
  tests stub the upstream — no live network calls in tests.

## Deviations

None.

## Lockfile

- Route: generated clean from birth in the build dir with no
  `node_modules` and no pre-existing lockfile:
  `npm install --package-lock-only --ignore-scripts --offline`
  (Route 3), then populated with `npm ci --ignore-scripts --offline`.
- Parses as JSON; `name` == `@reneturcios/sailguide`.
- Package entries: **128**.
- sha256: `ffe01ae50578ad1349d048676abbe8766433ad1d26028ac155d71ca2e9d94722`
  (re-verified unchanged after the subsequent offline `npm ci`).

## server.json

- Description exactly: "Dream On BVI charter concierge reads: fact
  sheet, availability, open windows, quotes, other yachts." —
  **99 characters**.
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/sailguide:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio. Dockerfile
  (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/sailguide"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/sailguide`
  and its smoke test asserts the server name `sailguide`.

## Verification (all run 2026-10-28)

1. `npx tsc --noEmit` → exit 0.
2. `npm run build` → exit 0; real `dist/` produced (cli, forward,
   index, mcp, types — .js, .d.ts, .js.map each).
3. `node dist/cli.js --version` → `0.1.0`.
4. `npm test` → **59 tests, 59 pass, 0 fail** (registration-integrity
   suite deep-equals the served tools/list against the fixture's 5
   tools, including the verbatim empty `required` arrays and the
   `requester` object shape, plus absence assertions for
   `request_hold` and `ask_owner` at registry, MCP-layer and CLI
   level, session handling/reuse, env override, the
   no-Authorization-header assertions under any environment,
   verbatim argument forwarding and result pass-through including
   upstream isError, SSE parsing, error rendering, MCP-layer
   validation, and CLI behaviour).
5. Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/dreamonsail-tools-original.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/sailguide/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/sailguide/parity-out` → **verdict PASS, parity 100.0**
   (5/5 identical, 0 missing, 0 extra, no schema diffs). Output in
   `parity-out/`.
