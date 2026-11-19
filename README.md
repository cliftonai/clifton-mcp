# <img src="assets/logo-square.png" alt="Clifton" width="32" height="32" align="middle"> Clifton MCP

[![Build](https://github.com/cliftonai/clifton-mcp/actions/workflows/ci.yml/badge.svg)](https://github.com/cliftonai/clifton-mcp/actions/workflows/ci.yml)
[![Version](https://img.shields.io/badge/version-0.1.3-blue.svg)](CHANGELOG.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![PRs welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/cliftonai/clifton-mcp/pulls)

Clifton exposes MCP tools for markets and finance research, raw structured data and document retrieval, and Data Vault file discovery and downloads. `clifton_ask` answers questions about public companies, ETFs, crypto, bonds, options, and macro data from SEC filings, earnings transcripts, analyst expectations, market data, news, and attached Clifton Data Vault files, then cites the documents or data it used.

![Clifton answering a financial research question in Claude](assets/demo.gif)


**Live plugin endpoint:** `https://ai.cliftonapi.com/v1/mcp`

**V2 preview endpoint (not live):** `https://ai.cliftonapi.com/v2/mcp`

**Release status:** these docs describe unreleased MCP v2. The existing MCP stays available while v2 is tested separately. Keep your existing connection until all v2 tools are tested and the cutover is approved. [Release notes](CHANGELOG.md#020---unreleased).

## Quick Start

`clifton_ask` on MCP v2 acknowledges a saved research request before its answer is ready. Your app waits `poll_after_seconds` (currently ten seconds), then uses `clifton_get_response` to read the current saved state. Each read returns as soon as storage completes; while running, your app waits again before its next read. This requires the app's own timing facility; hosts without one remain unvalidated for this flow. After the approved cutover, update existing connections to `/v2/mcp` and reconnect. [Upgrade and protocol details](API.md).

`model` selects `instant` (the default) or `deep-thinking` when enabled for an OAuth caller. `effort` independently selects `extra_low`, `low`, `medium` (the default), `high`, `extra_high`, or `ultra`. API-key callers use Instant with any of the six effort levels. Saved responses report both choices; resuming the same message requires that saved pair and the returned IDs. [Model, effort, and continuation details](API.md#clifton_ask-starts-one-answer).

### V2 host setup — validation pending

The browser/Desktop flows below have not passed v2 host acceptance; these are preview setup notes, not a supported-host claim. Do not replace an existing live connection. In Claude Desktop or Web, add a custom connector. Use ChatGPT Web's custom-plugin setup where your account permits it; ChatGPT Desktop has not been verified. Use this server URL:

```text
https://ai.cliftonapi.com/v2/mcp
```

Sign in with your Clifton account for OAuth connections. ChatGPT Web requires an eligible account and workspace permission.

**Step-by-step setup for each host →** [INSTALL.md](INSTALL.md)

### Claude Code Plugin

```
/plugin marketplace add cliftonai/clifton-mcp
/plugin install clifton-mcp@clifton-mcp
```

The packaged plugin remains on v1 until the separately approved live cutover. On first use, Claude Code connects to Clifton over OAuth. No Client ID is required. Then ask:

> What was NVDA's most recent reported quarterly revenue?

The plugin adds a [`clifton-research` skill](plugins/clifton-mcp/skills/clifton-research/SKILL.md) for the current v1 ask and Vault-listing contract. The six-tool reference below describes v2 preview behavior.

### Cursor, Codex, Manual Claude Code

Use an API key from [console.cliftonapi.com](https://console.cliftonapi.com) and pass it as:

```text
X-API-Key: <your-api-key>
```

Config snippets live in [`examples/`](examples/).

## Tools

- **`clifton_ask`** — starts a saved markets or finance request and returns its session/message IDs.
- **`clifton_get_response`** — reads the current saved state for those exact IDs, with the answer and citations when complete. Stop polling on completion or when action is required.
- **`clifton_search`** — explicit raw structured-data queries through the public search API.
- **`clifton_retrieve`** — explicit ranked document retrieval through the public retrieve API.
- **`clifton_list_vault_files`** — finds authorized Clifton Data Vault files. Use returned file IDs to download a file or pass them to `clifton_ask.attachments` for analysis.
- **`clifton_get_vault_file`** — returns an authorized signed S3 download URL for one file ID, valid for up to 15 minutes.

MCP v2 does not expose agent tools. Manage recurring agents and reports in the [Clifton console](https://console.cliftonapi.com). [Agent-tool availability](AGENT-TOOLS.md).

Full tool reference, auth, and errors: [API.md](API.md).

For the two direct-data tools, discover the contract at runtime: call with
`{"request_json":"{}"}` for public scopes, then with a returned scope in the
JSON string for request schemas, and build the query from that response.
Scopes and schemas come from the endpoint; MCP does not copy their catalog.
The selected tool's `request_json` instructions explain its request envelope.
These tools require an eligible organization account. General analysis and
sourced research still use `clifton_ask`. Direct-data delivery is bounded at
45 seconds and 1 MiB for the complete encoded response. Use native pagination
where supported or a narrower request, and never automatically repeat an
unknown outcome. [Direct data reference](API.md#direct-public-data).

## Get an account

Sign up at [console.cliftonapi.com](https://console.cliftonapi.com) for an API key and trial. Learn more at [cliftonai.com](https://cliftonai.com).

## Docs

- [INSTALL.md](INSTALL.md) — per-host setup
- [API.md](API.md) — endpoint, auth, tools, errors
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) — common connection/OAuth/tool issues
- [SUPPORT.md](SUPPORT.md) — how to get help
- [SECURITY.md](SECURITY.md) — vulnerability reporting
- [CHANGELOG.md](CHANGELOG.md) — changes to the tool surface

## Legal

[Privacy Policy](https://cliftonai.com/#/privacy) · [Terms of Service](https://cliftonai.com/#/terms)
