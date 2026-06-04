# Troubleshooting

Checks for common MCP setup failures. For protocol-level errors, see the **Troubleshooting** table in [API.md](API.md).

## Quick answers

### "Couldn't register with the sign-in service"

The OAuth sign-in handshake between the host and Clifton didn't complete. Confirm the connector URL is exactly `https://ai.cliftonapi.com/v1/mcp` and retry — this is sometimes transient. If it persists, it's more likely a Clifton-side issue than your setup: contact [support](SUPPORT.md) with the host you're using and the reference id shown in the error (e.g. `ofid_…`).

### Tool call returns `401 Unauthorized`

The API key is invalid, or the OAuth session expired. Create a new key in the [Clifton console](https://console.cliftonapi.com), or re-authenticate through the host's MCP connection flow.

### `clifton_ask` returns `clifton_ask_timeout`

The question needed more time than the client-safe budget allows. The response includes a Clifton-console link — open it to continue the question there — or ask a narrower question. If you contact support, quote the `request_id` from the response.

### A tool argument is rejected, or your host shows an option Clifton no longer has

Your host cached an older copy of Clifton's tool list. When Clifton updates a tool (adds or removes an argument), the host keeps using its cached version until it reconnects — so it may keep sending an argument the server now rejects, or keep offering one that's gone. **Reconnect Clifton to refresh the tool list, then retry:**

- **Claude Code:** run `/mcp`, reconnect Clifton — or restart Claude Code. (Plugin install: `/plugin uninstall clifton-mcp@clifton-mcp` then re-install also forces a clean fetch.)
- **Claude Web / Desktop:** open **Connectors**, toggle Clifton **off and on** (or remove and re-add it).
- **Cursor:** restart Cursor, or toggle the Clifton server off/on in **Settings → MCP**.
- **Codex:** restart Codex.

A plain reconnect re-fetches the current tool list on the next session. If the stale option still appears after reconnecting, remove and re-add the server.

### Something else

Email `support@cliftonai.com` with `[MCP]` in the subject. Include the host, the question, and the `request_id` from the response if one was shown.
