# Cursor — config snippet

Upgrading from v1? Change an existing `/v1/mcp` URL to `/v2/mcp` and reconnect instead of adding a second connection.

API key. `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "clifton": {
      "url": "https://ai.cliftonapi.com/v2/mcp",
      "headers": {
        "X-API-Key": "paste-your-key-here"
      }
    }
  }
}
```

Full setup: [../INSTALL.md#cursor](../INSTALL.md#cursor)
