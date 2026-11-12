# Claude Code — config snippets

Plugin (recommended — OAuth, no key):

```text
/plugin marketplace add cliftonai/clifton-mcp
/plugin install clifton-mcp@clifton-mcp
```

Manual OAuth:

```bash
claude mcp add --transport http \
  clifton https://ai.cliftonapi.com/v2/mcp
```

Run `/mcp` in Claude Code and select Clifton to sign in. Choose either the plugin or the manual server configuration.

Manual API key — `~/.claude.json`:

```json
{
  "mcpServers": {
    "clifton": {
      "type": "http",
      "url": "https://ai.cliftonapi.com/v2/mcp",
      "headers": { "X-API-Key": "paste-your-key-here" }
    }
  }
}
```

Full setup: [../INSTALL.md#claude-code](../INSTALL.md#claude-code)

## Direct public data

Ordinary research uses `clifton_ask`. For an explicit raw-data request, use
`clifton_search`; for ranked documents, use `clifton_retrieve`.

Discover each tool's current contract rather than keeping a local scope list:

1. Call the selected tool with `{"request_json":"{}"}` to discover public scopes.
2. Choose a returned scope and call with
   `{"request_json":"{\"scope\":\"<returned scope>\"}"}` for its schemas.
3. Build the native query from the returned schema and pass its JSON unchanged
   as `request_json`.

Eligible organization access is required. Discovery retains HTTP 422 with
`discovery:true`; actual data requires 2xx, no discovery flag and complete
JSON. Large successful output may be in a host file; read it to verify the
complete result. Use the native schema's pagination controls where supported.
Do not automatically repeat an unknown outcome.
