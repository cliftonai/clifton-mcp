# Clifton MCP — Developer Reference

Technical reference for connecting an MCP host to Clifton. Use this for Claude Code, Claude Desktop, Claude Web, Codex, Cursor, ChatGPT, or a custom MCP client.

> Looking for end-user setup ("paste this URL into Claude")? See [INSTALL.md](INSTALL.md).

---

## Endpoint

`https://ai.cliftonapi.com/v2/mcp`

- **Transport:** Streamable HTTP (MCP `2025-06-18`; `2025-03-26` accepted for client back-compat).
- **Protocol:** JSON-RPC 2.0 single-request per POST. Batch arrays are **not** supported.
- **Region:** us-east-2.

The public `tools/list` snapshot is published at `https://ai.cliftonapi.com/.well-known/mcp-tools.json`. OAuth-only tools are returned by authenticated `tools/list` after sign-in.

---

## Authentication

Clifton accepts OAuth bearer tokens and API keys. Follow the per-host instructions in [INSTALL.md](INSTALL.md) for your app's setup.

### API key (Claude Code manual, Cursor, Codex)

Header: `X-API-Key: <your-api-key>`

API keys are organization-scoped; the org's existing chat-session entitlements apply. Issue keys from the [Clifton console](https://console.cliftonapi.com).

### OAuth 2.1 (Claude, ChatGPT, and Codex)

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
claude mcp add --transport http clifton https://ai.cliftonapi.com/v2/mcp \
  --header "X-API-Key: $CLIFTON_API_KEY"
```

### Claude Web / Claude Desktop (OAuth)

Add a **Custom Connector** with URL `https://ai.cliftonapi.com/v2/mcp`. Claude discovers the authorization server via the protected-resource metadata, registers itself via DCR, and runs the OAuth flow. The host opens Clifton sign-in when authorization is required.

> If you're using a Team workspace, Custom Connectors are managed under **Organization settings → Connectors**, not personal Customize. Only the org owner can add connectors for a Team.

### Cursor

`~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "clifton": {
      "url": "https://ai.cliftonapi.com/v2/mcp",
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
[mcp_servers.clifton]
url = "https://ai.cliftonapi.com/v2/mcp"
http_headers = { "X-API-Key" = "paste-your-key-here" }
```

---

## Available tools

> **Tool contract:** `https://ai.cliftonapi.com/.well-known/mcp-tools.json` lists public tools. Authenticated `tools/list` applies user and organization gates and is authoritative for the caller. The summary below is non-authoritative and may lag the live contract.

### `clifton_ask` *(starts one answer)*

Submit a markets or finance question. Clifton saves the turn before returning its real session/message identifiers. The answer continues after this MCP response closes. Do not treat the acknowledgement as the answer.

| Input | Type | Required | Notes |
| --- | --- | --- | --- |
| `message` | string | yes | Question or task. |
| `effort` | `instant` or `research` | no | Defaults to Instant. Research requires OAuth and server activation for the caller. |
| `session_id` | string | no | Exact returned session ID for a follow-up; omit on the first call. |
| `message_id` | string | resume only | Exact returned message ID when answering `needs_input` or resuming `paused`. Omit for an ordinary new follow-up. |
| `response_format` | object | no | Feature-gated JSON Schema envelope. Resend on a continuation that requires structured output. |
| `data_sources` | string[] | no | Selected sources. |
| `attachments` | string[] | no | Authorized Clifton Data Vault file IDs found with `clifton_list_vault_files`. |
| `context` | object | no | Host context; never identity or authority. |

MCP v2 does not accept `incognito`. Unknown or foreign session identifiers are not permission to read another conversation.

A successful acknowledgement has `structuredContent`:

```json
{
  "session_id": "server-issued-session-id",
  "message_id": "server-issued-message-id",
  "status": "running",
  "complete": false,
  "poll_after_seconds": 10,
  "request_id": "support-correlation-id"
}
```

Use only returned identifiers. If the connection fails before you receive them, report an uncertain outcome with the request reference when available. **Do not invent IDs or automatically resubmit the question:** a new ask can create a second task and charge.

### `clifton_get_response` *(read-only)*

Pass the exact `session_id` and `message_id` from the acknowledgement. The same authenticated user and organization, or the same API-key identity, must fetch the result. Another user or key in the same organization cannot read it through this tool.

Call the tool immediately after acknowledgement. A running poll waits server-side for up to ten seconds, so the host does not need a sleep tool or an additional delay. Polling never starts, resumes, or charges for another answer.

| `status` | `complete` | What the host does |
| --- | --- | --- |
| `running` | false | Call `clifton_get_response` again with the same IDs. |
| `completed` | true | Use the saved answer. |
| `needs_input` | false | Stop polling; relay `question` and answer it using `clifton_ask` with the same IDs and effort. |
| `paused` | false | Stop polling; explicitly resume with `clifton_ask` using the same IDs and effort. |
| `failed`, `expired`, `stopped` | true | Stop polling and report the outcome. |

Completed responses include plain text in `content[].text`, raw `structuredContent.answer`, parsed `citations`, and optional `result` for structured output and `attachments` for produced files. Public source disclosures remain separate text blocks. While running, there is no final answer yet; token-by-token partial text is not returned.

Errors set `isError` and include `error.code`, `class`, and `retryable` in the first JSON content block. Errors with a saved terminal state also include the typed `structuredContent`; errors without a valid saved handle omit it. Input-schema validation errors instead use a first JSON block with top-level `code: "invalid_arguments"` and `validation_errors`. Missing, deleted, or inaccessible turns produce `response_not_found`; do not interpret that as permission to resubmit. `execution_outcome_unknown` means the invocation ended without a saved terminal result. A storage-limit failure is explicit and never presented as a complete answer or valid JSON result. Database read failures remain retryable polling errors.

### Direct public data

Use `clifton_search` for explicit raw structured data through public
`POST /v3/search`, and `clifton_retrieve` for ranked document retrieval through
public `POST /v3/retrieve`. Analysis and ordinary sourced answers continue to
use `clifton_ask`. Both direct tools require eligible organization access;
availability also depends on environment rollout.

Both accept one required field, `request_json`, a string containing the exact
native public request JSON (at most 256 KiB UTF-8). Keep numeric literals in
that string unchanged. Authentication supplies identity and payer; these are
not request fields.

Discover the current native contract through the same selected tool:

1. Send `{"request_json":"{}"}` to discover public scopes.
2. Choose a returned scope and send
   `{"request_json":"{\"scope\":\"<returned scope>\"}"}` to discover its request schemas.
3. Build the native query from those schemas and send its JSON as `request_json`.

Apply the selected tool's request protocol:

- `clifton_search`: put the selected public scope and the entry's `version`
  at the root. Build `request` with its selector from the entry's `type`
  metadata and the remaining fields described by `entry.request`.
- `clifton_retrieve`: `entry.request` describes the complete root JSON body.

The endpoint determines the verb. Selector values and field definitions come
from API discovery; fill those fields with the values requested by the user.

There is no MCP copy of the scope, schema, operation or filter catalog.
Pagination uses only the native controls shown by the returned schema. MCP
does not create cursors, fetch another page automatically or assume every
operation supports pagination.

A public response exposes `http_status`, `content_type`, and `response_json`
in `structuredContent`, with optional `retry_after` and `discovery:true`.
`response_json` is the complete native JSON string. The first text block is
that same exact JSON; the second is JSON status metadata for hosts that expose
only content. It does not duplicate `response_json`.

A pure original empty or scope-only probe that returns public
`422 invalid_input` with nonempty scope/schema education is successful MCP
discovery: `isError:false`, `http_status:422`, `discovery:true`. That result is
schema education, not query data. Ordinary query data requires 2xx, no discovery
flag and a complete valid body. Actual query validation, access/rate refusals
and service failures remain errors, even if they contain help text.

The complete encoded JSON-RPC response or SSE message must fit within 1 MiB,
including framing, metadata and both body copies. Actual errors have an 8 KiB
aggregate text limit. Overflow replaces the whole result with an explicit
error; JSON is never silently truncated. A host may put a large successful
result in a file: retrieve the complete file or narrow the request using the
native schema's controls. The server limit is not a promise that every host
shows the result inline.

Delivery ends within 45 seconds of native entry, including authentication and
setup. An already dispatched operation remains retained through its original
completion/accounting. An unknown outcome must not be automatically retried.
These calls have no saved job/session/message handle and are not polled with
`clifton_get_response`. Public search has no metered debit; retrieve uses the
existing admission and debits its actual provider cost once.

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

Five tools for creating, deleting, and reading Clifton agents (standing monitors that generate reports when their condition fires). Full guide with examples, deletion behavior, the agent limit, `output_mode`, and `owner_email` semantics: [AGENT-TOOLS.md](AGENT-TOOLS.md).

| Tool | Summary | Key input fields |
|---|---|---|
| `clifton_create_agent` | Multi-turn agent creation; commits on your confirmation. API-key access requires an organization gate and an authorized `owner_email` | `message` (required), `session_id`, `output_mode` (`chat` \| `models`), `owner_email`* |
| `clifton_delete_agent` | Delete one agent owned by an authorized user. Existing reports remain available | `agent_id` (required), `owner_email`* |
| `clifton_list_agents` | List agents, newest first | `owner_email`*, `limit` (default 10), `offset` |
| `clifton_list_agent_reports` | List report summaries, newest first | `owner_email`*, `agent_id`, `limit` (default 10), `offset` |
| `clifton_get_agent_report` | One report: markdown (chat), spreadsheet link (models), or a no-report `reason` | report id, `owner_email`* |

\* `owner_email` defaults to the signed-in user for OAuth callers. API-key callers must provide an authorized manager email in the key's organization. `clifton_create_agent` also requires Clifton to enable API-key creation for that organization.

---

## Execution and limits

- `effort` selects the existing Instant or Research model. Research remains gated; discover the schema and eligibility through authenticated `tools/list`.
- MCP does not impose its former short execution cap. MCP Instant uses the selected model maximum when it raises the ordinary default (currently 300 seconds); Research retains its configured default (currently 720 seconds). These configured values remain subject to cost, available credit, cumulative message budgets and the Router invocation deadline. Other callers' defaults remain unchanged.
- Acknowledgement and final completion are separate. Losing the host connection after acknowledgement does not itself stop the Router's retained execution.
- Questions and pauses require action; polling alone cannot finish them. A resume preserves the original message's cumulative budget.
- The per-organization chat quota still applies to submissions. Response reads do not reserve or pay for a new answer.
- Keep the response-reading tool available while disabling new submissions so accepted work can drain.
- JSON-RPC batch arrays are not supported.

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `401 Unauthorized` immediately on any RPC | Missing or wrong `X-API-Key` / `Authorization` header. | Verify the key is current; for OAuth, complete the auth flow before issuing tool calls. |
| `mcp_upgrade_required` from the old endpoint | The connection uses `/v1/mcp`. | Update it to `https://ai.cliftonapi.com/v2/mcp`, then reconnect and refresh tools. |
| A submission returns `status: running` | The request is accepted and saved. | Poll `clifton_get_response` using its exact session and message IDs. |
| Tool not found on `tools/call` (`METHOD_NOT_FOUND`) | Tool not in the currently-enabled set. | Confirm via `tools/list`; only currently-enabled tools respond. |
| OAuth completes but `tools/call` returns 401 | The access token's identity claim could not be resolved. | Re-run sign-in; if persistent, contact support with the trace details. |
| `Automatic client registration isn't supported` in Claude Web | Client tried DCR against the wrong endpoint. | Update the connector URL to `https://ai.cliftonapi.com/v2/mcp`; the DCR shim handles registration. |
| Connector "redirect_mismatch" during OAuth | The sign-in callback was rejected. | Reconnect Clifton. If it persists, contact support with your app name and the error text. |

For unresolved issues, include the `X-Request-Id` from the failing response and the host you were using.

---

## Further reading

- Public tool contract: `https://ai.cliftonapi.com/.well-known/mcp-tools.json`
- OAuth protected-resource metadata: `https://ai.cliftonapi.com/.well-known/oauth-protected-resource`
- Customer setup walkthrough: [INSTALL.md](INSTALL.md)
- Common host issues: [TROUBLESHOOTING.md](TROUBLESHOOTING.md)
