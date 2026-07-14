# Clifton MCP — Developer Reference

Technical reference for connecting an MCP host to Clifton. Use this for Claude Code, Claude Desktop, Claude Web, Codex, Cursor, ChatGPT, or a custom MCP client.

> Looking for end-user setup ("paste this URL into Claude")? See [INSTALL.md](INSTALL.md).

---

## Endpoint

`https://ai.cliftonapi.com/v1/mcp`

- **Transport:** Streamable HTTP (MCP `2025-06-18`; `2025-03-26` accepted for client back-compat).
- **Protocol:** JSON-RPC 2.0 single-request per POST. Batch arrays are **not** supported.
- **Region:** us-east-2.

The public `tools/list` snapshot is published at `https://ai.cliftonapi.com/.well-known/mcp-tools.json`. OAuth-only tools are returned by authenticated `tools/list` after sign-in.

---

## Authentication

Use OAuth for Claude Code plugin installs, Claude Web, Claude Desktop, and supported ChatGPT workspaces. Use an API key for Claude Code manual setup, Cursor, Codex, or custom hosts that cannot run OAuth/DCR.

### API key (Claude Code manual, Cursor, Codex)

Header: `X-API-Key: <your-api-key>`

API keys are organization-scoped; the org's existing chat-session entitlements apply. Issue keys from the [Clifton console](https://console.cliftonapi.com).

### OAuth 2.1 (Claude Code plugin, Claude Web, Claude Desktop, ChatGPT)

Standard OAuth 2.1 with PKCE. Clifton's authorization endpoints support Dynamic Client Registration, so MCP clients can register and exchange tokens without manual Client ID entry.

| Endpoint | URL |
|---|---|
| Protected-resource metadata | `https://ai.cliftonapi.com/.well-known/oauth-protected-resource` (RFC 9728) |
| Authorization-server metadata | `https://ai.cliftonapi.com/.well-known/oauth-authorization-server` (RFC 8414) |
| Dynamic Client Registration | `POST https://ai.cliftonapi.com/oauth/register` (RFC 7591) |
| Authorize | `https://ai.cliftonapi.com/oauth/authorize` |
| Token | `https://ai.cliftonapi.com/oauth/token` |

Tokens are JWTs. Pass as `Authorization: Bearer <jwt>`. Audience is `client_id`; identity (`username`, `organization_id`) is resolved server-side from the user's profile.

PKCE-only (no client secrets). Refresh tokens are accepted at the token endpoint.

---

## Per-host setup

### Claude Code CLI

**Plugin (OAuth):**

```text
/plugin marketplace add cliftonai/clifton-mcp
/plugin install clifton-mcp@clifton-mcp
```

No API key is required. The plugin ships the Clifton MCP server config, and Claude Code registers through OAuth/DCR on first use.

**Or, manual with an API key:**

```bash
claude mcp add --transport http clifton https://ai.cliftonapi.com/v1/mcp \
  --header "X-API-Key: $CLIFTON_API_KEY"
```

### Claude Web / Claude Desktop (OAuth)

Add a **Custom Connector** with URL `https://ai.cliftonapi.com/v1/mcp`. Claude discovers the authorization server via the protected-resource metadata, registers itself via DCR, and runs the OAuth flow. The host opens Clifton sign-in when authorization is required.

> If you're using a Team workspace, Custom Connectors are managed under **Organization settings → Connectors**, not personal Customize. Only the org owner can add connectors for a Team.

### Cursor

`~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "clifton": {
      "url": "https://ai.cliftonapi.com/v1/mcp",
      "headers": {
        "X-API-Key": "paste-your-key-here"
      }
    }
  }
}
```

### Codex CLI

`~/.codex/config.toml`:

```toml
[features]
rmcp_client = true   # required for streamable-HTTP MCP servers

[mcp_servers.clifton]
url = "https://ai.cliftonapi.com/v1/mcp"
http_headers = { "X-API-Key" = "paste-your-key-here" }
```

---

## Available tools

> **Tool contract:** `https://ai.cliftonapi.com/.well-known/mcp-tools.json` lists public tools. Authenticated `tools/list` applies user and organization gates and is authoritative for the caller. The summary below is non-authoritative and may lag the live contract.

### `clifton_ask` *(read-only)*

Questions about markets and finance — public companies, tickers, sectors, indexes, ETFs, crypto, bonds, options, and macro/economic data — plus filings, earnings, news, and financial metrics. Use it for extraction, comparison, calculation, and explanations with citations.

**Input**

| Field | Type | Required | Notes |
|---|---|---|---|
| `message` | string | yes | The question or task. |
| `session_id` | string | no | Echo back from a prior response to continue a conversation. |
| `incognito` | boolean | no | Skip persistence. |
| `data_sources` | string[] | no | Restrict retrieval to a subset of sources. |
| `attachments` | string[] | no | Clifton Data Vault file IDs to include. Use `clifton_list_vault_files` to find accessible IDs. |
| `context` | object | no | Host-supplied metadata. |

**Output**

| Field | Type | Notes |
|---|---|---|
| `content[].text` | string | Plain-text answer with citation markup stripped, ready to display. |
| `structuredContent.answer` | string | Raw answer with inline `<citation>` markup preserved. |
| `structuredContent.citations` | array | Parsed citation objects (`url`, `text`, `snippet`). |
| `structuredContent.session_id` | string \| null | Server-acknowledged session id. Echo to continue. |
| `structuredContent.request_id` | string | Present on errors. Identifies the call in Clifton's logs; quote it when contacting support. |

### `clifton_list_vault_files` *(read-only)*

Find Clifton Data Vault files for an authorized owner, then pass returned file IDs to `clifton_ask.attachments`.

OAuth callers default to the signed-in user. API-key callers must pass an `owner_email` for a manager in the authenticated organization.

**Input**

| Field | Type | Required | Notes |
|---|---|---|---|
| `query` | string | no | Search by filename and indexed file content when available. Omit to browse folders. Max 100 characters. |
| `folder_path` | string | no | Folder path to browse or constrain search. Defaults to `/`. |
| `storage_class` | `EPHEMERAL` \| `DURABLE` | no | Optional storage class filter. |
| `limit` | integer | no | Defaults to 50; maximum 1000. |
| `offset` | integer | no | Zero-based pagination offset. |
| `owner_email` | string | API key only | Vault owner. OAuth defaults to the signed-in user; API keys require an authorized manager email. |

**Output**

| Field | Type | Notes |
|---|---|---|
| `files` | array | File metadata including `file_id`, filename, content type, size, scope, folder, and timestamps. |
| `folders` | array | Immediate child folders when browsing. Empty when `query` is provided. |
| `attachment_ids` | string[] | File IDs from `files[].file_id`; pass these to `clifton_ask.attachments`. |
| `current_path` | string | Folder path used for the result. |
| `pagination` | object | `limit`, `offset`, `total`, `count`, `has_more`. |

### Agent tools

Four tools for creating and reading Clifton agents (standing monitors that generate reports when their condition fires). Full guide with examples, the agent limit, `output_mode`, and `owner_email` semantics: [AGENT-TOOLS.md](AGENT-TOOLS.md).

| Tool | Summary | Key input fields |
|---|---|---|
| `clifton_create_agent` | Multi-turn agent creation; commits on your confirmation. **OAuth only** — not offered to API-key callers | `message` (required), `session_id`, `output_mode` (`chat` \| `models`) |
| `clifton_list_agents` | List agents, newest first | `owner_email`*, `limit` (default 10), `offset` |
| `clifton_list_agent_reports` | List report summaries, newest first | `owner_email`*, `agent_id`, `limit` (default 10), `offset` |
| `clifton_get_agent_report` | One report: markdown (chat), spreadsheet link (models), or a no-report `reason` | report id, `owner_email`* |

\* `owner_email` defaults to the signed-in user for OAuth callers; required for API-key callers and must belong to the key's organization. Cross-organization reads are rejected.

---

## Operational Notes (Not Contract)

These notes describe current server behavior and are **not part of the contract**. Check the well-known endpoint and server changelog for contract changes.

- **Single tier.** `clifton_ask` runs one retrieval tier; there is no tier or mode argument to set. (A longer research tier may return in a later version.)
- **Latency.** Typically 20–40 seconds; quick lookups (a ticker, one reported metric) are faster. A question that needs more than the instant tier comes back promptly with a continue-in-console link rather than running longer.
- **MCP-client-safe time budget.** A call that can't finish within the budget returns `clifton_ask_timeout` with a link to continue the question in the Clifton console and a `request_id` for support.
- **Per-org chat quota** applies to `clifton_ask` (same path as the Clifton web console's chat).
- **No batch JSON-RPC arrays** — one request per POST.
- **No partial answer chunks** in `tools/call` responses; the answer arrives as one completed result.

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `401 Unauthorized` immediately on any RPC | Missing or wrong `X-API-Key` / `Authorization` header. | Verify the key is current; for OAuth, complete the auth flow before issuing tool calls. |
| `404 Not Found` on `POST /v1/mcp` | Endpoint disabled for this host. | Confirm the URL is `https://ai.cliftonapi.com/v1/mcp`; contact support if it 404s. |
| `tools/call` returns `clifton_ask_timeout` | The question needed more time than the MCP-client-safe budget allows. | Open the Clifton-console link in the response to continue it there, or narrow the question. Quote the `request_id` if you contact support. |
| Tool not found on `tools/call` (`METHOD_NOT_FOUND`) | Tool not in the currently-enabled set. | Confirm via `tools/list`; only currently-enabled tools respond. |
| OAuth completes but `tools/call` returns 401 | The access token's identity claim could not be resolved. | Re-run sign-in; if persistent, contact support with the trace details. |
| `Automatic client registration isn't supported` in Claude Web | Client tried DCR against the wrong endpoint. | Update the connector URL to `https://ai.cliftonapi.com/v1/mcp`; the DCR shim handles registration. |
| Connector "redirect_mismatch" during OAuth | The host's OAuth callback URL is not pre-registered. | Major Claude / OpenAI / Cursor callbacks are already registered. For other hosts, contact support to add the callback URL. |

For unresolved issues, include the `X-Request-Id` from the failing response and the host you were using.

---

## Further reading

- Public tool contract: `https://ai.cliftonapi.com/.well-known/mcp-tools.json`
- OAuth protected-resource metadata: `https://ai.cliftonapi.com/.well-known/oauth-protected-resource`
- Customer setup walkthrough: [INSTALL.md](INSTALL.md)
- Common host issues: [TROUBLESHOOTING.md](TROUBLESHOOTING.md)
