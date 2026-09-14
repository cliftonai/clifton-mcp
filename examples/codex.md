# Codex CLI and Desktop

## OAuth

Add Clifton and sign in:

```bash
codex mcp add clifton --url https://ai.cliftonapi.com/v1/mcp
codex mcp login clifton
```

For manual configuration, add this to `~/.codex/config.toml`, then run `codex mcp login clifton`:

```toml
[mcp_servers.clifton]
url = "https://ai.cliftonapi.com/v1/mcp"
```

Update an existing `clifton` entry instead of adding a duplicate.

## API key

To use a Clifton API key, add the header to the same entry:

```toml
[mcp_servers.clifton]
url = "https://ai.cliftonapi.com/v1/mcp"
http_headers = { "X-API-Key" = "paste-your-key-here" }
```

Restart Codex after saving. Current versions need no `rmcp_client` feature flag.

Full setup: [Codex](../INSTALL.md#codex) · [ChatGPT Desktop](../INSTALL.md#chatgpt-desktop). Web setup is separate.
