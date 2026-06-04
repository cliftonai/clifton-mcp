# Codex CLI — config snippet

API key. `~/.codex/config.toml`:

```toml
[features]
rmcp_client = true   # enables streamable-HTTP MCP servers

[mcp_servers.clifton]
url = "https://ai.cliftonapi.com/v1/mcp"
http_headers = { "X-API-Key" = "paste-your-key-here" }
```

Full setup: [../INSTALL.md#codex](../INSTALL.md#codex)
