# Changelog

Public changes to the Clifton MCP tool surface.

## [Unreleased]

## [0.1.1] - 2026-06-10

- **Agent tools.** `clifton_create_agent` (OAuth sign-in required) creates a recurring Clifton agent through a multi-turn conversation, with optional `output_mode: chat | models`. Three read tools — `clifton_list_agents`, `clifton_list_agent_reports`, `clifton_get_agent_report` — list your agents and fetch their reports; available to both OAuth and API-key callers (API-key callers pass `owner_email`). Guide: [AGENT-TOOLS.md](AGENT-TOOLS.md).
- **OAuth sign-in for Claude Code.** The server accepts loopback redirect URIs and Clifton sign-in registers the fixed callback port `8765`; the plugin pins `oauth.callbackPort: 8765` in its config. Manual setup: `claude mcp add --transport http --callback-port 8765 clifton https://ai.cliftonapi.com/v1/mcp`.
- If a connected host doesn't show the new tools, reconnect it — see [TROUBLESHOOTING.md](TROUBLESHOOTING.md) (an app restart alone is not enough on Claude Desktop).

## [0.1.0]

- Initial public docs + Claude Code plugin (`clifton-mcp`).
