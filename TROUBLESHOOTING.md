# Troubleshooting

For current live access use [the v1 connection guide](LIVE.md). V2-specific guidance below applies only after live cutover or against an explicitly identified test endpoint.

Checks for common MCP setup failures. For protocol-level errors, see the **Troubleshooting** table in [API.md](API.md).

## Quick answers

### "Couldn't register with the sign-in service"

The OAuth sign-in handshake between the host and Clifton didn't complete. Confirm that the connector URL matches your intended endpoint: current live `https://ai.cliftonapi.com/v1/mcp`, or `/v2/mcp` only after cutover. Authenticate again — this is sometimes transient. If it persists, it's more likely a Clifton-side issue than your setup: contact [support](SUPPORT.md) with the host you're using and the reference id shown in the error (e.g. `ofid_…`).

### OAuth sign-in does not complete

Keep the current live server URL `https://ai.cliftonapi.com/v1/mcp`; use `/v2/mcp` only after the announced cutover. Reauthenticate the intended existing connection. Complete sign-in in the browser and return to your app.

For Claude Code, use `/mcp` to authenticate. For Codex CLI, run `codex mcp login clifton`. If the error persists, contact [support](SUPPORT.md) with your app name and the error text. Do not send credentials or the full sign-in URL.

### Tool call returns `401 Unauthorized`

The API key is invalid, or the OAuth session expired. Create a new key in the [Clifton console](https://console.cliftonapi.com), or re-authenticate through the host's MCP connection flow.

### V1 ask times out

A v1 `clifton_ask_timeout` means no answer arrived within that client request budget. Open the returned Clifton console link to continue, or narrow the question. Do not switch to unreleased v2 as a timeout workaround.

### The connection says an upgrade is required

This section applies after the live v2 cutover, not to the current v1 plugin.

Change its URL from `/v1/mcp` to `https://ai.cliftonapi.com/v2/mcp`, then reconnect and refresh its tools. Plugin users should update `clifton-mcp` and restart or reconnect it. Do not add duplicate Clifton connections.

### The request says it is running

This is an acknowledgement, not the final answer. Wait `poll_after_seconds` (currently ten seconds), then call `clifton_get_response` with the exact returned session and message IDs. Each call reads the current saved state and returns as soon as storage completes. While the status remains `running`, wait the recommended delay before reading again. Your host needs its own timing facility; hosts without one remain unvalidated for this flow. Stop polling on completion, failure, expiry, or a stopped request. A question or pause needs action, so stop polling and follow the returned instructions.

If the initial acknowledgement itself was lost, do not automatically submit the question again. Report the uncertain outcome and request reference instead; a fresh submission can create another task and charge.

### Your host shows fewer or older Clifton tools than expected, or rejects a tool argument

Your host cached an older copy of Clifton's tool list, or the tool is not enabled for your account. After an approved v2 upgrade, a stale cache may omit `clifton_get_vault_file` or still offer removed agent tools. V2 has six tools: ask, get response, search, retrieve, list Vault files, and get Vault file. **Reconnect Clifton to refresh the tool list:**

- **Claude Code:** run `/mcp`, reconnect Clifton — or restart Claude Code. (Plugin install: `/plugin uninstall clifton-mcp@clifton-mcp` then re-install also forces a clean fetch.)
- **Claude Web / Desktop:** open **Connectors**, toggle Clifton **off and on** (or remove and re-add it). Restarting the app alone is **not** enough — the tool list is tied to the connector, not the app session.
- **Cursor:** restart Cursor, or toggle the Clifton server off/on in **Settings → MCP**.
- **Codex:** restart Codex.

A plain reconnect re-fetches the current tool list on the next session. If the stale option still appears after reconnecting, remove and re-add the server.

### A Vault download link has expired

Call `clifton_get_vault_file` again with the same `file_id` to request a fresh authorized URL. Signed URLs are valid for up to 15 minutes and can stop working sooner. If the tool returns `vault_file_unavailable`, the file is missing, expired, not ready, or inaccessible; no download URL is returned. Keep the file ID rather than an old signed URL as the lasting reference.

### A data result appears clipped

The 1 MiB response limit governs what Clifton sends. Your host may show its model a smaller preview, even when its client log contains the complete result. Increasing a client output setting does not guarantee complete model input.

If the host reports a complete result-file path, read that file and verify the whole JSON. Otherwise, use the API-discovered page/limit controls to request a smaller result, or use the REST API for larger data. Do not treat a preview as the full data. [Direct data reference](API.md#direct-public-data).

### Something else

Email `support@cliftonai.com` with `[MCP]` in the subject. Include the host, the question, and the `request_id` from the response if one was shown.
