# Live connection

The live Clifton MCP endpoint is:

```text
https://ai.cliftonapi.com/v2/mcp
```

Setup for each app is in [INSTALL.md](INSTALL.md). The tool reference is in [API.md](API.md), and the overview is in the [README](README.md).

## Claude Code plugin

```text
/plugin marketplace add cliftonai/clifton-mcp
/plugin install clifton-mcp@clifton-mcp
```

Use `/mcp` to sign in. Ask a finance question and confirm that `clifton_ask` returns session and message IDs and `clifton_get_response` later returns a completed, sourced answer.

## Upgrading from v1

The v1 endpoint is retired and only returns an upgrade notice. Change an existing connection's URL to `/v2/mcp`, then reconnect and refresh its tools. Plugin users update `clifton-mcp` to 0.2.0 and restart or reconnect. Do not add a second Clifton connection.

Agent tools are not available in v2. Use the [Clifton console](https://console.cliftonapi.com) for recurring agents and reports.
