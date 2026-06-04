# Claude Code — config snippets

Plugin (recommended — OAuth, no key):

```text
/plugin marketplace add Clifton-Software-Inc/clifton-mcp
/plugin install clifton-mcp@clifton-mcp
```

Manual OAuth — `~/.claude.json`:

```json
{
  "mcpServers": {
    "clifton": {
      "type": "http",
      "url": "https://ai.cliftonapi.com/v1/mcp"
    }
  }
}
```

Manual API key — `~/.claude.json`:

```json
{
  "mcpServers": {
    "clifton": {
      "type": "http",
      "url": "https://ai.cliftonapi.com/v1/mcp",
      "headers": { "X-API-Key": "paste-your-key-here" }
    }
  }
}
```

Full setup: [../INSTALL.md#claude-code](../INSTALL.md#claude-code)
