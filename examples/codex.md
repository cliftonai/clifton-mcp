# Codex CLI and Desktop — config snippet

API key. `~/.codex/config.toml`:

```toml
[mcp_servers.clifton]
url = "https://ai.cliftonapi.com/v1/mcp"
http_headers = { "X-API-Key" = "paste-your-key-here" }
```

Restart Codex after saving. Current versions need no `rmcp_client` feature flag.

Full setup: [Codex](../INSTALL.md#codex) · [ChatGPT Desktop](../INSTALL.md#chatgpt-desktop). Web setup is separate.
