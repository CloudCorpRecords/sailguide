# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the hosted Dream On charter concierge MCP server. Labelled subset: 5 of the 7 captured tools are answered locally for tools/list, and every tools/call is forwarded upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when the upstream issues one).
- Tools: `get_dream_on_facts`, `check_availability`, `find_open_windows`, `get_quote`, `suggest_other_yachts`. The two captured send tools are omitted (see BUILD-REPORT.md). `get_dream_on_facts` and `suggest_other_yachts` carry an empty `required` array in the capture, served verbatim.
- Upstream results pass through verbatim, including the isError flag; upstream failures surface as `sailguide error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). The upstream endpoint can be overridden via `DREAMONSAIL_MCP_URL`. The upstream record declares no headers, so no auth header is ever sent.
