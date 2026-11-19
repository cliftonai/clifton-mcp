# Cursor — config snippet

**V2 PREVIEW ONLY:** do not apply this v2 configuration before live cutover. For current access use [the live v1 connection](../LIVE.md). Host acceptance remains pending unless explicitly recorded.

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
