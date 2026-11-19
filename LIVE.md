# Live v1 connection

The packaged plugin currently uses `https://ai.cliftonapi.com/v1/mcp`. The six-tool v2 reference is a preview; do not replace this live address until the live cutover is announced.

## Claude Code plugin

```text
/plugin marketplace add cliftonai/clifton-mcp
/plugin install clifton-mcp@clifton-mcp
```

Use `/mcp` to sign in. Ask a finance question and confirm `clifton_ask` returns its answer. The current v1 plugin does not use the v2 `clifton_get_response` flow.

## Manual connection

Use `https://ai.cliftonapi.com/v1/mcp` as the remote HTTP endpoint in a host that supports it. Authenticate with OAuth, or supply your existing Clifton API key through the host's secret/header configuration. Follow your workspace's connector permissions; a host's Connected badge alone is not a successful answer test.

For exact v1 arguments and its historical per-host setup, see the [pinned v1 API reference](https://github.com/cliftonai/clifton-mcp/blob/346c08995539fea38a5192835dcb69d9ee4e172c/API.md). Those older host-specific screens may differ from your installed version. The [v2 installation guide](INSTALL.md) is for a later cutover, not current setup.

Use the Clifton console for recurring agents. This plugin does not advertise or invoke MCP agent-management tools.
