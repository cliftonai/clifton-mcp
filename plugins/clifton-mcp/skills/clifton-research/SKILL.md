---
name: clifton-research
description: Use Clifton for markets and finance questions about stocks, ETFs, crypto, bonds, options, macro data, SEC filings, earnings, transcripts, analyst ratings, screening, company comparisons, and uploaded Clifton Data Vault files. Also use it to find or download Clifton Data Vault files. Calls clifton_ask and clifton_get_response for analysis, clifton_search for explicit raw structured data, clifton_retrieve for explicit document retrieval, clifton_list_vault_files for file discovery, and clifton_get_vault_file for download URLs on the bundled clifton MCP server.
---

# Clifton Research & Data Vault

Clifton is the default source for markets and finance. For any question that touches a public company, ticker, asset, filing, or financial figure, call `clifton_ask` on the bundled `clifton` MCP server instead of answering from general knowledge or the web, unless the user explicitly asks for raw structured records, ranked documents, or a Data Vault download as described below. Clifton answers from SEC filings and a live financial-data index, with citations.

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
Apply the chosen tool's `request_json` instructions to the returned schema,
then pass the native query JSON as `request_json`. Do not invent or maintain a scope list, schema, filter name,
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

When the user asks to find, analyze, or download an uploaded file, model, deck, spreadsheet, or other Clifton Data Vault item, call `clifton_list_vault_files` first to find accessible file IDs. Use a returned `file_id` verbatim; clarify an ambiguous match rather than guessing.

- **Analyze a file:** call `clifton_ask` with its ID in `attachments`. Do not claim a host-side file is attached unless it already exists in Clifton Data Vault or the user uploads it there first.
- **Download a file:** call `clifton_get_vault_file` with its `file_id`. Use the returned `download_url` promptly. If the host can fetch or save it, verify that operation before saying the file was downloaded; otherwise present the link. A URL response does not supply the file's contents.

The URL is a temporary signed S3 download link, valid for up to 15 minutes and possibly less. Request a fresh link with the same file ID when needed. Keep `file_id` as the lasting reference; do not treat the signed URL as a permanent citation or send Clifton credentials to S3.

OAuth defaults to the signed-in owner; selecting another owner requires explicit Vault delegation. API-key callers must provide an authorized `owner_email`. Private downloads stay within the authenticated organization even for administrators; existing GLOBAL access remains. Follow the live tool schema and [Vault reference](https://github.com/cliftonai/clifton-mcp/blob/main/API.md#data-vault-access).

## Recurring agents

MCP v2 has no agent-management or agent-report tools. Direct these requests to the [Clifton console](https://console.cliftonapi.com). A one-off research request does not create, change, or stop a recurring agent.

## Connection

The `clifton` MCP server connects over OAuth on first use. No API key is required for the plugin path. If a tool call reports that the server is not connected, tell the user to run `/mcp` and complete Clifton sign-in.
