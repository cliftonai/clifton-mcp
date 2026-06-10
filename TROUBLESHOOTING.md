# Troubleshooting

Checks for common MCP setup failures. For protocol-level errors, see the **Troubleshooting** table in [API.md](API.md).

## Quick answers

### "Couldn't register with the sign-in service"

The OAuth sign-in handshake between the host and Clifton didn't complete. Confirm the connector URL is exactly `https://ai.cliftonapi.com/v1/mcp` and retry — this is sometimes transient. If it persists, it's more likely a Clifton-side issue than your setup: contact [support](SUPPORT.md) with the host you're using and the reference id shown in the error (e.g. `ofid_…`).

### Claude Code OAuth fails with `redirect_uri not permitted` / `redirect_mismatch`

Claude Code's loopback callback port didn't match the one Clifton's sign-in expects. Clifton requires the exact redirect `http://localhost:8765/callback`; by default Claude Code picks a random port (you'll see something like `http://localhost:61674/callback` in the browser URL), which is rejected.

- **Plugin path:** the plugin ships a pinned port. Confirm it applied with `claude mcp get clifton` — it should show `OAuth: callback_port 8765`. If it doesn't (or shows a random port), use the manual command below instead of the plugin's server.
- **Manual `claude mcp add` path:** you added the server without the fixed port. Remove and re-add it with the flag, then re-authenticate:

  ```bash
  claude mcp remove clifton
  claude mcp add --transport http --callback-port 8765 \
    clifton https://ai.cliftonapi.com/v1/mcp
  ```

  Run `/mcp` to sign in. A random port in the browser URL means the port wasn't pinned.

### Tool call returns `401 Unauthorized`

The API key is invalid, or the OAuth session expired. Create a new key in the [Clifton console](https://console.cliftonapi.com), or re-authenticate through the host's MCP connection flow.

### `clifton_ask` returns `clifton_ask_timeout`

The question needed more time than the client-safe budget allows. The response includes a Clifton-console link — open it to continue the question there — or ask a narrower question. If you contact support, quote the `request_id` from the response.

### Your host shows fewer or older Clifton tools than expected, or rejects a tool argument

Your host cached an older copy of Clifton's tool list. When Clifton releases a new tool or updates an existing one, the host keeps using its cached version until it reconnects — newly released tools (such as the agent tools) won't appear, and a changed argument may be rejected or a removed one still offered. **Reconnect Clifton to refresh the tool list, then retry:**

- **Claude Code:** run `/mcp`, reconnect Clifton — or restart Claude Code. (Plugin install: `/plugin uninstall clifton-mcp@clifton-mcp` then re-install also forces a clean fetch.)
- **Claude Web / Desktop:** open **Connectors**, toggle Clifton **off and on** (or remove and re-add it). Restarting the app alone is **not** enough — the tool list is tied to the connector, not the app session.
- **Cursor:** restart Cursor, or toggle the Clifton server off/on in **Settings → MCP**.
- **Codex:** restart Codex.

A plain reconnect re-fetches the current tool list on the next session. If the stale option still appears after reconnecting, remove and re-add the server.

### Something else

Email `support@cliftonai.com` with `[MCP]` in the subject. Include the host, the question, and the `request_id` from the response if one was shown.
