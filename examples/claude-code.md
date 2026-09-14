# Claude Code — config snippets

Plugin (recommended — OAuth, no key):

```text
/plugin marketplace add cliftonai/clifton-mcp
/plugin install clifton-mcp@clifton-mcp
```

Manual OAuth:

```bash
claude mcp add --transport http \
  clifton https://ai.cliftonapi.com/v1/mcp
```

Run `/mcp` in Claude Code and select Clifton to sign in. Choose either the plugin or the manual server configuration.

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
