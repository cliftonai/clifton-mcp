# Troubleshooting

Checks for common MCP setup failures. For protocol-level errors, see the **Troubleshooting** table in [API.md](API.md).

## Quick answers

### "Couldn't register with the sign-in service"

The OAuth sign-in handshake between the host and Clifton didn't complete. Confirm the connector URL is exactly `https://ai.cliftonapi.com/v1/mcp` and retry — this is sometimes transient. If it persists, it's more likely a Clifton-side issue than your setup: contact [support](SUPPORT.md) with the host you're using and the reference id shown in the error (e.g. `ofid_…`).

### OAuth sign-in does not complete

Confirm the server URL is `https://ai.cliftonapi.com/v1/mcp`, then disconnect Clifton and reconnect. Complete sign-in in the browser and return to your app.

For Claude Code, use `/mcp` to authenticate. For Codex CLI, run `codex mcp login clifton`. If the error persists, contact [support](SUPPORT.md) with your app name and the error text. Do not send credentials or the full sign-in URL.

### Tool call returns `401 Unauthorized`

The API key is invalid, or the OAuth session expired. Create a new key in the [Clifton console](https://console.cliftonapi.com), or re-authenticate through the host's MCP connection flow.

### `clifton_ask` returns `clifton_ask_timeout`

The question needed more time than the client-safe budget allows. The response includes a Clifton-console link — open it to continue the question there — or ask a narrower question. If you contact support, quote the `request_id` from the response.

### Your host shows fewer or older Clifton tools than expected, or rejects a tool argument

Your host cached an older copy of Clifton's tool list. When Clifton releases a new tool or updates an existing one, the host keeps using its cached version until it reconnects — newly released tools such as `clifton_list_vault_files` or the agent tools won't appear, and a changed argument may be rejected or a removed one still offered. **Reconnect Clifton to refresh the tool list, then retry:**

- **Claude Code:** run `/mcp`, reconnect Clifton — or restart Claude Code. (Plugin install: `/plugin uninstall clifton-mcp@clifton-mcp` then re-install also forces a clean fetch.)
- **Claude Web / Desktop:** open **Connectors**, toggle Clifton **off and on** (or remove and re-add it). Restarting the app alone is **not** enough — the tool list is tied to the connector, not the app session.
- **Cursor:** restart Cursor, or toggle the Clifton server off/on in **Settings → MCP**.
- **Codex:** restart Codex.

A plain reconnect re-fetches the current tool list on the next session. If the stale option still appears after reconnecting, remove and re-add the server.

### Something else

Email `support@cliftonai.com` with `[MCP]` in the subject. Include the host, the question, and the `request_id` from the response if one was shown.
