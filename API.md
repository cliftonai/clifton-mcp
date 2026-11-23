# Clifton MCP — Developer Reference

Technical reference for connecting an MCP host to Clifton. Use this for Claude Code, Claude Desktop, Claude Web, Codex, Cursor, ChatGPT, or a custom MCP client.

> Looking for end-user setup ("paste this URL into Claude")? See [INSTALL.md](INSTALL.md).

---

## Endpoint

`https://ai.cliftonapi.com/v2/mcp`

This is the live MCP v2 endpoint. The former `/v1/mcp` endpoint is retired: `initialize` reports the server as `Clifton (upgrade required)` with instructions to switch to `/v2/mcp`, and every tool call returns `mcp_upgrade_required` with an `upgrade_url`. It runs no tools or models. To upgrade, change the connection URL to `/v2/mcp`, reconnect, and refresh tools; plugin users update `clifton-mcp` to 0.2.0. See the [release notes](CHANGELOG.md#020---2026-11-19).

- **Transport:** Streamable HTTP (MCP `2025-06-18`; `2025-03-26` accepted for client back-compat).
- **Protocol:** JSON-RPC 2.0 single-request per POST. Batch arrays are **not** supported.
- **Region:** us-east-2.

Authenticated `tools/list` on the endpoint is authoritative for each caller.

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
| Protected-resource metadata | `https://ai.cliftonapi.com/.well-known/oauth-protected-resource/v2/mcp` (RFC 9728) |
| Authorization-server metadata | `https://ai.cliftonapi.com/.well-known/oauth-authorization-server` (RFC 8414) |
| Dynamic Client Registration | `POST https://ai.cliftonapi.com/oauth/register` (RFC 7591) |
| Authorize | `https://ai.cliftonapi.com/oauth/authorize` |
| Token | `https://ai.cliftonapi.com/oauth/token` |

Requested scopes are `openid`, `email`, and `profile`. Tokens are JWTs. Pass as `Authorization: Bearer <jwt>`. Audience is `client_id`; identity (`username`, `organization_id`) is resolved server-side from the user's profile.

PKCE-only (no client secrets). Refresh tokens are accepted at the token endpoint.

---

## Per-host setup

### Claude Code CLI

**Plugin (OAuth):**

```text
/plugin marketplace add cliftonai/clifton-mcp
/plugin install clifton-mcp@clifton-mcp
```

The plugin connects to `https://ai.cliftonapi.com/v2/mcp`. No API key is required; Claude Code registers through OAuth/DCR on first use.

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

V2 serves seven tools: `clifton_ask`, `clifton_get_response`, `clifton_search`, `clifton_retrieve`, `clifton_list_vault_files`, `clifton_get_vault_file`, and `clifton_upload_vault_file`. A caller may see fewer because of access or rollout gates. Agent tools are unavailable in v2; use the [Clifton console](https://console.cliftonapi.com) for agents and reports.

### `clifton_ask` *(starts one answer)*

Submit a markets or finance question. Clifton saves the turn before returning its real session/message identifiers. The answer continues after this MCP response closes. Do not treat the acknowledgement as the answer.

| Input | Type | Required | Notes |
| --- | --- | --- | --- |
| `message` | string | yes | Question or task. |
| `model` | `instant` or `deep-thinking` | no | Defaults to `instant`. Production currently offers `instant` only; `deep-thinking` appears in `tools/list` when it is enabled. Deep Thinking requires OAuth and server activation for the caller. API-key callers are Instant-only. |
| `effort` | `extra_low`, `low`, `medium`, `high`, or `extra_high` | no | Defaults to `medium`. MCP does not offer `ultra`. Each depth applies within each permitted model. |
| `session_id` | string | no | Exact returned session ID for a follow-up; omit on the first call. |
| `message_id` | string | resume only | Exact returned message ID when answering `needs_input` or resuming `paused`. Omit for an ordinary new follow-up. |
| `response_format` | object | no | Feature-gated JSON Schema envelope. Resend on a continuation that requires structured output. |
| `data_sources` | string[] | no | Selected sources. |
| `attachments` | string[] | no | Authorized Clifton Data Vault file IDs found with `clifton_list_vault_files` or returned by your own finalized `clifton_upload_vault_file` uploads. Requires `model: "deep-thinking"`. |
| `context` | object | no | Host context; never identity or authority. |

Model and effort are independent: Instant with `extra_high` remains Instant; Deep Thinking with `extra_low` remains Deep Thinking. Both fields are optional strings, not nullable. Omitting either applies its default anew, rather than inheriting a previous turn's choice. Use the exact wire spellings above: `model: "research"`, the old `effort: "instant"` or `effort: "research"`, and spaced or provider-specific effort names are not accepted by v2. Refresh discovery when upgrading from v1.

MCP v2 does not accept `incognito`. Unknown or foreign session identifiers are not permission to read another conversation.

A successful acknowledgement has `structuredContent`:

```json
{
  "session_id": "server-issued-session-id",
  "message_id": "server-issued-message-id",
  "model": "instant",
  "effort": "medium",
  "status": "running",
  "complete": false,
  "poll_after_seconds": 10,
  "request_id": "support-correlation-id"
}
```

Use only returned identifiers. If the connection fails before you receive them, report an uncertain outcome with the request reference when available. **Do not invent IDs or automatically resubmit the question:** a new ask can create a second task and charge.

The acknowledgement saves and returns the resolved `model` and `effort`, including their defaults. To resume any saved turn in `needs_input` or `paused`, resend that exact saved pair, its returned `session_id` and `message_id`, and the original `response_format` if one was supplied. Omitting the fields requests Instant/medium; it does not recover the paused turn's settings. A different pair is refused before a new execution. A completed follow-up uses `session_id` without `message_id`, starts a new turn, and may choose a different pair.

An ask refusal sets `isError:true`. Read machine-readable details from the first JSON text content block rather than assuming a successful `structuredContent` receipt exists. Argument errors include `code: "invalid_arguments"` and `validation_errors`; other bounded refusals carry their code and any details in that JSON block.

### `clifton_get_response` *(read-only)*

Pass the exact `session_id` and `message_id` from the acknowledgement. The same authenticated user and organization, or the same API-key identity, must fetch the result. Another user or key in the same organization cannot read it through this tool.

This tool accepts IDs only; it does not take model or effort arguments. A recognized owned turn echoes its saved `model` and `effort` in running, action-needed, and terminal responses. Previously saved MCP turns whose choices cannot be identified omit both fields and cannot authorize a resume; do not guess the missing choices. Ordinary non-MCP turns and missing or foreign IDs remain unavailable through this reader.

Each call makes one bounded logical read of the exact saved turn and returns as soon as storage completes, without an intentional polling delay. `poll_after_seconds` (currently `10`) is the recommended client delay: wait about ten seconds after acknowledgement and between reads while the status is `running`. The client needs its own timing facility to honor that delay; hosts without one remain unvalidated for this flow. Polling never starts, resumes, or charges for another answer.

| `status` | `complete` | What the host does |
| --- | --- | --- |
| `running` | false | Wait `poll_after_seconds` seconds, then call `clifton_get_response` again with the same IDs. |
| `completed` | true | Stop polling; use the saved answer. |
| `needs_input` | false | Stop polling; relay `question` and answer it using `clifton_ask` with the same IDs, saved model/effort pair, and original `response_format` if supplied. |
| `paused` | false | Stop polling; explicitly resume with `clifton_ask` using the same IDs, saved model/effort pair, and original `response_format` if supplied. |
| `failed`, `expired`, `stopped` | true | Stop polling and report the outcome. |

Completed responses include plain text in `content[].text`, raw `structuredContent.answer`, parsed `citations`, and optional `result` for structured output and `attachments` for produced files. Public source disclosures remain separate text blocks. While running, there is no final answer yet; token-by-token partial text is not returned.

Errors set `isError`; inspect the first JSON text content block for machine-readable details. Saved-state failures include `error.code`, `class`, and `retryable`, and a saved terminal state also includes typed `structuredContent`. Refusals without a valid saved handle omit it. Input-schema validation errors use a first JSON block with top-level `code: "invalid_arguments"` and `validation_errors`. Missing, deleted, or inaccessible turns produce `response_not_found`; do not interpret that as permission to resubmit. `execution_outcome_unknown` means the invocation ended without a saved terminal result. A storage-limit failure is explicit and never presented as a complete answer or valid JSON result. Database read failures remain retryable polling errors.

### Direct public data

| Tool | Public endpoint |
| --- | --- |
| `clifton_search` | `/v3/search` |
| `clifton_retrieve` | `/v3/retrieve` |

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

### Data Vault access

All three Vault tools accept OAuth and API keys. Listing and download share the existing Vault feature gate (`mcp_list_vault_files`); upload uses `mcp_vault_upload`. Environment rollout can enable each tool independently. Existing admission requires a positive organization credit balance and applies rate limits, even though the tools have no per-call unit charge.

OAuth callers default to the signed-in user. An explicit `owner_email` for another owner requires `vault:delegate`; ordinary organization-manager status or workflow delegation does not grant it. API-key callers must supply `owner_email`; without `vault:delegate` it must name an authorized organization manager. Vault delegation permits another selected owner within the authenticated organization. No Vault tool accepts an organization override.

Private downloads stay within the authenticated organization, including for administrators and Vault delegates. Authorized owner selection permits another owner's files inside that organization; it does not bypass the organization check. Existing direct-ID access to `GLOBAL` files in `READY` state remains available across organizations. The separate REST Vault API keeps its existing administrator behavior.

### `clifton_list_vault_files` *(read-only)*

Find authorized Clifton Data Vault files. Pass a returned `file_id` to `clifton_get_vault_file` to download it, or to `clifton_ask.attachments` for analysis. Listing returns metadata and IDs, not download URLs or file contents.

**Input**

| Field | Type | Required | Notes |
|---|---|---|---|
| `query` | string | no | Search by filename and indexed file content when available. Omit to browse folders. Max 100 characters. |
| `folder_path` | string | no | Folder path to browse or constrain search. Defaults to `/`. |
| `storage_class` | `EPHEMERAL` \| `DURABLE` | no | Optional storage class filter. |
| `limit` | integer | no | Defaults to 50; maximum 1000. |
| `offset` | integer | no | Zero-based pagination offset. |
| `owner_email` | string | API-key calls | Authorized owner selector, subject to the Vault access rules above. OAuth defaults to the signed-in user. |

**Output**

| Field | Type | Notes |
|---|---|---|
| `files` | array | File metadata including `file_id`, `original_filename`, content type, size, scope, folder, and timestamps. |
| `folders` | array | Immediate child folders when browsing. Empty when `query` is provided. |
| `attachment_ids` | string[] | File IDs from `files[].file_id`; pass these to `clifton_ask.attachments`. |
| `current_path` | string | Folder path used for the result. |
| `is_system_folder` | boolean | Whether the current path is a system folder. |
| `pagination` | object | `limit`, `offset`, `total`, `count`, `has_more`. |

### `clifton_get_vault_file` *(read-only)*

Request a download link for one authorized Vault file. Use a `file_id` returned by listing or a saved Clifton artifact; do not invent an ID or pass an S3 address.

**Input**

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `file_id` | UUID string | yes | Exact Vault file ID. |
| `owner_email` | string | API-key calls | Authorized owner selector, subject to the Vault access rules above. OAuth defaults to the signed-in user. |

No other input fields are accepted. Callers cannot choose a bucket, object key, URL lifetime, organization, or administrator override.

**Output**

All five fields appear in one JSON text block and the identical `structuredContent` object:

| Field | Type | Notes |
| --- | --- | --- |
| `file_id` | string | The requested Vault file ID. |
| `download_url` | string | Authorized HTTPS S3 URL for downloading the file. |
| `filename` | string | Original filename. |
| `content_type` | string | Stored content type, or `application/octet-stream`. |
| `expires_in` | integer | Nominal URL lifetime: `900` seconds. |

Download promptly with an HTTPS GET to `download_url`; S3 needs no Clifton bearer token or API-key header. The response contains a link, not file bytes. A host must observe a successful fetch or save before claiming the file was downloaded; hosts without that ability can present the link.

Signed URLs are transferable bearer links, valid for up to 15 minutes and sometimes less. Access is checked when the URL is issued; an issued link may remain usable until expiry after access changes. Call the tool again for a fresh authorized URL. Keep `file_id` as the lasting reference rather than saving the signed URL as a permanent citation. Private Vault downloads do not use public short URLs.

A missing, expired, non-ready, or inaccessible file produces the same `vault_file_unavailable` refusal with `isError:true` and no download URL. Owner, schema, gate, credit, and rate-limit failures remain their existing MCP refusals. Database or S3 failures remain service failures rather than being reported as a missing file. Check the MCP result, not HTTP success alone.

### `clifton_upload_vault_file` *(write)*

Put a local file into Clifton Data Vault so `clifton_ask` can analyze it. Files already in Vault need no upload; find them with `clifton_list_vault_files`. Upload needs a host that can make an HTTP PUT request, such as Claude Code or Codex through its shell. Hosts without HTTP PUT, including Claude Web and ChatGPT, should ask the user to upload the file in the [Clifton console](https://console.cliftonapi.com).

Upload takes three steps.

**1. Start.** Call the tool without `file_id`.

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `filename` | string | yes | Original filename with a supported extension. |
| `file_size` | integer | yes | Exact size in bytes; at most 50 MiB (52,428,800 bytes). |
| `content_type` | string | no | Defaults to the MIME type for the filename extension. |
| `storage_class` | `EPHEMERAL` \| `DURABLE` | no | Defaults to `EPHEMERAL`, which expires after 24 hours. Use `DURABLE` to keep the file. |
| `folder_path` | string | no | Vault folder. Defaults to `/`. |
| `owner_email` | string | API-key calls | Authorized owner selector, subject to the Vault access rules above. OAuth defaults to the signed-in user. |

The result contains `file_id`, `upload_url`, `method: "PUT"`, `headers` (`Content-Type` and `Content-Length`), `expires_in: 900`, and `next`.

**2. PUT the bytes.** Send the exact raw file bytes to `upload_url` with the returned headers within 15 minutes. Do not use multipart or chunked transfer; a wrong length fails the signed upload. S3 needs no Clifton bearer token or API-key header.

```sh
curl -X PUT -H "Content-Type: <returned type>" -H "Content-Length: <returned length>" --data-binary @file "<upload_url>"
```

**3. Finalize.** After the PUT succeeds, call the tool again with only `file_id`, plus the same `owner_email` if you supplied one. Finalizing checks the file contents whatever Content-Type was declared and returns `file_id`, `filename`, `status: "ready"`, and `attachment_ids`. The file is usable only once it is ready. Calling finalize again for the same ready file is safe.

Pass the returned `attachment_ids` to `clifton_ask.attachments` with `model: "deep-thinking"`; Instant does not read files. Only your own uploads can be attached. A delegated upload belongs to the selected owner and cannot be attached to the caller's `clifton_ask`.

Errors set `isError:true`; read the code from the first JSON text content block.

| Code | Meaning | What to do |
| --- | --- | --- |
| `upload_not_uploaded` | Finalize was called before the bytes arrived. | PUT the bytes, then finalize again before the URL expires. |
| `upload_expired` | The upload URL expired unused. | Start again without `file_id`. Bytes uploaded before expiry can still be finalized. |
| `upload_size_mismatch` | The uploaded bytes do not match `file_size`. | Start again with the exact size. |
| `upload_not_found` | The `file_id` is unknown, belongs to someone else, or is in an unusable state. | Start again. |
| `invalid_file` | Unsupported extension or contents, or the file contains macros. | Upload a supported file. |
| `upload_rejected` | Another upload refusal. | Report it to the user. |
| `vault_storage_limit_exceeded` | The owner's Vault storage limit is reached. | Delete unneeded Vault files, then start again. |

Upload has its own organization feature gate (`mcp_vault_upload`), separate from the listing and download gate. Like the other Vault tools, it requires a positive organization credit balance and applies rate limits.

---

## Execution and limits

- `model` selects Instant or Deep Thinking; `effort` selects a standard depth within that model. Defaults are Instant/medium. Production currently serves Instant only. Deep Thinking remains OAuth-only and gated; discover the schema and eligibility through authenticated `tools/list`. MCP does not offer `ultra` effort.
- MCP does not impose its former short execution cap. MCP Instant uses the selected model maximum when it raises the ordinary default (currently 300 seconds); Deep Thinking retains its configured default (currently 720 seconds). These configured values remain subject to cost, available credit, cumulative message budgets and the Router invocation deadline. Changing effort does not grant a larger time or cost allowance. Other callers' defaults remain unchanged.
- Acknowledgement and final completion are separate. Losing the host connection after acknowledgement does not itself stop the Router's retained execution.
- Questions and pauses require action; polling alone cannot finish them. A same-message resume must match the saved model/effort pair and preserves the original message's cumulative budget. Omitted values apply the defaults, not the saved choices.
- The per-organization chat quota still applies to submissions. Response reads do not reserve or pay for a new answer.
- Keep the response-reading tool available while disabling new submissions so accepted work can drain.
- JSON-RPC batch arrays are not supported.

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `401 Unauthorized` immediately on any RPC | Missing or wrong `X-API-Key` / `Authorization` header. | Verify the key is current; for OAuth, complete the auth flow before issuing tool calls. |
| `mcp_upgrade_required` from the old endpoint | The connection uses the retired `/v1/mcp`. | Update it to `https://ai.cliftonapi.com/v2/mcp`, then reconnect and refresh tools. Plugin users update `clifton-mcp` to 0.2.0. |
| A submission returns `status: running` | The request is accepted and saved. | Wait `poll_after_seconds` seconds, then call `clifton_get_response` using its exact session and message IDs. |
| Tool not found on `tools/call` (`METHOD_NOT_FOUND`) | Tool not in the currently-enabled set. | Confirm via `tools/list`; only currently-enabled tools respond. |
| OAuth completes but `tools/call` returns 401 | The access token's identity claim could not be resolved. | Re-run sign-in; if persistent, contact support with the trace details. |
| `Automatic client registration isn't supported` in Claude Web | Client tried DCR against the wrong endpoint. | Use `https://ai.cliftonapi.com/v2/mcp`; its DCR shim handles registration. |
| Connector "redirect_mismatch" during OAuth | The sign-in callback was rejected. | Reconnect Clifton. If it persists, contact support with your app name and the error text. |

For unresolved issues, include the `X-Request-Id` from the failing response and the host you were using.

---

## Further reading

- OAuth protected-resource metadata: `https://ai.cliftonapi.com/.well-known/oauth-protected-resource/v2/mcp`
- Customer setup walkthrough: [INSTALL.md](INSTALL.md)
- Common host issues: [TROUBLESHOOTING.md](TROUBLESHOOTING.md)
