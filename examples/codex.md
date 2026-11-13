# Codex CLI and Desktop

## OAuth

Add Clifton and sign in:

```bash
codex mcp add clifton --url https://ai.cliftonapi.com/v2/mcp
codex mcp login clifton
```

For manual configuration, add this to `~/.codex/config.toml`, then run `codex mcp login clifton`:

```toml
[mcp_servers.clifton]
url = "https://ai.cliftonapi.com/v2/mcp"
```

Update an existing `clifton` entry instead of adding a duplicate.

## API key

To use a Clifton API key, add the header to the same entry:

```toml
[mcp_servers.clifton]
url = "https://ai.cliftonapi.com/v2/mcp"
http_headers = { "X-API-Key" = "paste-your-key-here" }
```

Restart Codex after saving. Current versions need no `rmcp_client` feature flag.

Full setup: [Codex](../INSTALL.md#codex) · [ChatGPT Desktop](../INSTALL.md#chatgpt-desktop). Web setup is separate.

## Direct public data

Ordinary research uses `clifton_ask`. For an explicit raw-data request, use
`clifton_search`; for ranked documents, use `clifton_retrieve`.

Discover each tool's current contract rather than keeping a local scope list:

1. Call the selected tool with `{"request_json":"{}"}` to discover public scopes.
2. Choose a returned scope and call with
   `{"request_json":"{\"scope\":\"<returned scope>\"}"}` for its schemas.
3. Build the native query from the returned schema and pass its JSON unchanged
   as `request_json`.

Follow the selected tool's `request_json` guidance for its request envelope.
[Protocol details](../API.md#direct-public-data) explain the two tools.

Eligible organization access is required. Discovery retains HTTP 422 with
`discovery:true`; actual data requires 2xx, no discovery flag and complete
JSON. Large successful output may be in a host file; read it to verify the
complete result. Use the native schema's pagination controls where supported.
Do not automatically repeat an unknown outcome.
