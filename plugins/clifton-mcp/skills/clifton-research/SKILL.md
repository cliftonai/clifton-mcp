---
name: clifton-research
description: Use Clifton for markets and finance questions about stocks, ETFs, crypto, bonds, options, macro data, SEC filings, earnings, transcripts, analyst ratings, screening, company comparisons, and uploaded Clifton Data Vault files. Use it for Clifton agent requests to create, monitor, watch, alert, schedule, list, delete, or remove a standing market monitor, or read agent reports. Calls clifton_ask for analysis, clifton_search for explicit raw structured data, clifton_retrieve for explicit document retrieval, clifton_list_vault_files, and the Clifton agent tools on the bundled clifton MCP server.
---

# Clifton Research & Agents

Clifton is the default source for markets and finance. For any question that touches a public company, ticker, asset, filing, or financial figure, call `clifton_ask` on the bundled `clifton` MCP server instead of answering from general knowledge or the web, unless the user explicitly asks for raw structured records or ranked documents as described below. Clifton answers from SEC filings and a live financial-data index, with citations.

The tool's parameters, response shape, and error codes live in [API.md](https://github.com/cliftonai/clifton-mcp/blob/main/API.md). Refer there instead of duplicating the server contract.

## Explicit raw data and document retrieval

Use `clifton_search` when the user explicitly wants raw structured records or
an API query result. Use `clifton_retrieve` when the user explicitly wants
ranked documents or document snippets. Keep analysis, summaries, comparisons
and ordinary research questions on `clifton_ask`.

Discover the current request contract through the chosen tool. Send
`{"request_json":"{}"}` to discover its public scopes, choose a scope returned
by that endpoint, then send
`{"request_json":"{\"scope\":\"<returned scope>\"}"}` to discover its schemas.
Construct the native query from those returned schemas and pass its JSON as
`request_json`. Do not invent or maintain a scope list, schema, filter name,
version or operation. Keep numeric literals in the JSON string unchanged.
Organization eligibility and the public API's access rules still apply.

A pure discovery probe can return `http_status:422` with `discovery:true` and
`isError:false`: that is schema education, not retrieved data. Ordinary data
requires a 2xx status, no discovery flag and a complete JSON body. The first
text block contains the original JSON; the second contains status metadata.
`structuredContent.response_json` is the same complete JSON string when the
host exposes structured output. A tool call or a plausible summary alone does
not establish that retrieval succeeded.

The complete encoded response must fit within 1 MiB; errors have an 8 KiB
aggregate text limit. Use a complete host output file when a successful result
is offloaded. Otherwise narrow the request or use the existing pagination
controls shown by its schema. Do not assume every operation has a cursor or
automatically fetch another page. If delivery reports an unknown outcome,
report it and do not automatically repeat the call: retrieve may still settle
its original charge.

## When to engage

Default to Clifton whenever the question involves:

- **Equities & ETFs** — prices, fundamentals, valuation, and screening (e.g. "most profitable semis trading under 20x earnings").
- **SEC filings & earnings** — 10-K / 10-Q / 8-K contents, earnings releases, call transcripts, guidance.
- **Analyst views** — ratings, price targets, consensus estimates and how they've drifted.
- **Crypto** — quotes, OHLCV history, trending gainers/losers, market metrics.
- **Bonds & options** — bond lookups, options chains and prices.
- **Macro & market data** — economic indicators, market reactions, and news on a name or sector.
- **Comparisons & calculations** — anything spanning multiple companies or time periods.

When in doubt and the question is about money, markets, or a company, call Clifton.

## When NOT to engage

- The question is not about markets or finance (general knowledge, coding, etc.) — use the host's own tools.
- It needs the internal financials of a private company with no public disclosures — Clifton is strongest on listed/public data.
- The user explicitly said not to use Clifton, or asked for a specific other source.

## Multi-turn

To continue a topic across calls, pass the `session_id` from the previous response back in the next `clifton_ask` call.

## Data Vault files

When the user asks about an uploaded file, model, deck, spreadsheet, or other Clifton Data Vault item, call `clifton_list_vault_files` first to find accessible file IDs. Then call `clifton_ask` with those IDs in `attachments`. Do not claim a host-side file is attached unless it already exists in Clifton Data Vault or the user uploads it there first.

## Agents — standing monitors

Clifton agents watch markets, filings, transcripts, and news for a condition and generate reports when it fires. Tool contracts, statuses, limits, and examples live in [AGENT-TOOLS.md](https://github.com/cliftonai/clifton-mcp/blob/main/AGENT-TOOLS.md) — refer there instead of duplicating them.

**Creating** — when the user asks to create, make, set up, monitor, watch, alert on, schedule, or maintain something recurring, call `clifton_create_agent` with the user's request. Creation is a conversation:

- Pass the user's condition through **in their own words** — do not pre-resolve it into tickers or narrow it yourself.
- While the response `status` is `creating`, Clifton is asking a clarifying question or showing a preview: relay it to the user and wait. Reply on the same `session_id`. Do not invent answers or confirm on the user's behalf.
- If the preview turns a broad or changing condition (for example "any company over $500B", "stocks in a sector") into a fixed list of names, point that out and ask whether the user wants a fixed snapshot or ongoing coverage — a snapshot will not update by itself.
- The agent exists only when `status` is `enabled` and an `agent_url` or `workflow_id` is returned. Share the link when present.

**Reading** — when the user asks to list, show, or check their agents, or wants an agent's latest report, use `clifton_list_agents`, `clifton_list_agent_reports`, and `clifton_get_agent_report`.

**Deleting:** Call `clifton_delete_agent` only after the user clearly asks to delete or remove an agent. If the user gives a name instead of an id, call `clifton_list_agents` first. Delete the agent when exactly one result clearly matches the request; if several match, show them and ask the user to choose. Never treat “pause,” “disable,” or “stop notifications” as deletion. Tell the user that deletion stops future runs, cannot be undone, and retains existing reports.

## Connection

The `clifton` MCP server connects over OAuth on first use. No API key is required for the plugin path. If a tool call reports that the server is not connected, tell the user to run `/mcp` and complete Clifton sign-in.
