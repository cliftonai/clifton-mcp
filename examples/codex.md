# Codex CLI and Desktop — config snippet

Codex supports OAuth, but Clifton sign-in currently fails with `redirect_mismatch` in a live client check. Use this API-key configuration until that issue is resolved. [OAuth troubleshooting](../TROUBLESHOOTING.md#chatgpt-desktop-or-codex-oauth-returns-redirect_mismatch)

`~/.codex/config.toml`:

```toml
[mcp_servers.clifton]
url = "https://ai.cliftonapi.com/v1/mcp"
http_headers = { "X-API-Key" = "paste-your-key-here" }
```

Restart Codex after saving. Current versions need no `rmcp_client` feature flag.

Full setup: [Codex](../INSTALL.md#codex) · [ChatGPT Desktop](../INSTALL.md#chatgpt-desktop). Web setup is separate.
