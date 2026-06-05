# <img src="assets/logo-square.png" alt="Clifton" width="32" height="32" align="absmiddle"> Clifton MCP

[![Build](https://github.com/cliftonai/clifton-mcp/actions/workflows/ci.yml/badge.svg)](https://github.com/cliftonai/clifton-mcp/actions/workflows/ci.yml)
[![Version](https://img.shields.io/badge/version-0.1.0-blue.svg)](CHANGELOG.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![PRs welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/cliftonai/clifton-mcp/pulls)

Clifton exposes one MCP tool, `clifton_ask`, for markets and finance research — public companies, ETFs, crypto, bonds, options, and macro data. It answers from SEC filings, earnings transcripts, analyst expectations, market data, and news, then cites the documents or data it used.

![Clifton answering a financial research question in Claude](assets/demo.gif)


**Endpoint:** `https://ai.cliftonapi.com/v1/mcp`

## Quick Start

### Claude Web, Claude Desktop, ChatGPT

In Claude, add a custom connector. In ChatGPT, create a custom MCP app in Developer mode. Use this MCP server URL:

```text
https://ai.cliftonapi.com/v1/mcp
```

Sign in with your Clifton account when prompted. In ChatGPT, custom MCP apps require Developer mode and a supported workspace; the Clifton OAuth flow has not been verified end to end there yet.

**Step-by-step setup for each host →** [INSTALL.md](INSTALL.md)

### Claude Code Plugin

```
/plugin marketplace add cliftonai/clifton-mcp
/plugin install clifton-mcp@clifton-mcp
```

On first use, Claude Code connects to Clifton over OAuth. No Client ID is required. Then ask:

> What was NVDA's most recent reported quarterly revenue?

The plugin also adds a [`clifton-research` skill](plugins/clifton-mcp/skills/clifton-research/SKILL.md) that tells Claude Code when to call `clifton_ask`.

### Cursor, Codex, Manual Claude Code

Use an API key from [console.cliftonapi.com](https://console.cliftonapi.com) and pass it as:

```text
X-API-Key: <your-api-key>
```

Config snippets live in [`examples/`](examples/).

## Tools

- **`clifton_ask`** — answers markets & finance questions (equities, ETFs, crypto, bonds, options, macro) from filings, earnings transcripts, analyst expectations, market data, and news; returns citations.

Full tool reference, auth, and errors: [API.md](API.md).

## Get an account

Sign up at [console.cliftonapi.com](https://console.cliftonapi.com) for an API key and trial. Learn more at [cliftonai.com](https://www.cliftonai.com).

## Docs

- [INSTALL.md](INSTALL.md) — per-host setup
- [API.md](API.md) — endpoint, auth, tools, errors
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) — common connection/OAuth/tool issues
- [SUPPORT.md](SUPPORT.md) — how to get help
- [SECURITY.md](SECURITY.md) — vulnerability reporting
- [CHANGELOG.md](CHANGELOG.md) — changes to the tool surface
