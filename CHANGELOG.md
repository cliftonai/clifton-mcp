# Changelog

Public changes to the Clifton MCP tool surface.

## [0.2.1] - 2026-11-23

- **Data Vault uploads.** `clifton_upload_vault_file` puts a local file into Data Vault in three steps: start with the filename and exact size, PUT the bytes to the returned signed URL, then finalize. Pass the returned `attachment_ids` to `clifton_ask` with Deep Thinking to analyze the file. Uploads default to 24-hour EPHEMERAL storage; files can be up to 50 MiB. Hosts need HTTP PUT, so Claude Code and Codex can upload while Claude Web and ChatGPT users upload in the Clifton console. See [API.md](API.md#clifton_upload_vault_file-write).
- **Plugin skill.** `clifton-research` now covers uploading a local file before analysis. Plugin users update `clifton-mcp` to 0.2.1 and reconnect to refresh the tool list.

## [0.2.0] - 2026-11-19

MCP v2 is live at `https://ai.cliftonapi.com/v2/mcp`, and the packaged plugin now targets it. The v1 endpoint is retired: it returns only an upgrade notice (`mcp_upgrade_required` with an `upgrade_url`) and runs no tools or models.

- **Upgrading.** Manual connections change their URL from `/v1/mcp` to `/v2/mcp`, then reconnect to refresh the tool list. Plugin users update `clifton-mcp` to 0.2.0 and restart or reconnect. Do not add a second Clifton connection.
- **Long-running answers.** `clifton_ask` returns saved session/message identifiers. `clifton_get_response` reads the current saved state and returns as soon as storage completes. Callers use their own timing facility to wait `poll_after_seconds` (currently ten seconds) after acknowledgement and between running reads. Polling never starts another answer or adds another answer charge.
- **Model and effort.** `model` chooses `instant` or `deep-thinking`; `effort` chooses `extra_low`, `low`, `medium`, `high`, or `extra_high`; MCP does not offer `ultra`. Defaults are Instant/medium. Production currently serves Instant only; Deep Thinking retains its OAuth access requirement and appears in `tools/list` when enabled. V2 rejects the former profile values in `effort` and does not accept `model: research`.
- **Continuation.** Acknowledgements and recognized saved responses return both choices. Resume a saved question or pause with the exact model/effort pair, returned IDs, and original `response_format` if supplied. Omission applies defaults anew, not inheritance. Stop polling when action is required.
- **Persistence required.** MCP v2 omits incognito. A lost initial acknowledgement must not trigger automatic resubmission. See [API.md](API.md).
- **Data Vault downloads.** Use `clifton_list_vault_files` to find an authorized file ID, then `clifton_get_vault_file` for a signed S3 download URL valid for up to 15 minutes. Use `clifton_ask.attachments` when the request is to analyze the file instead.
- **Vault owner permissions.** OAuth defaults to the signed-in user; selecting another owner requires `vault:delegate`, not ordinary manager status or workflow delegation. API-key calls require an authorized owner selector. Private downloads remain inside the authenticated organization even for administrators; existing GLOBAL-file access remains.
- **Six-tool scope.** V2 exposes ask, get response, search, retrieve, list Vault files, and get Vault file. Agent creation, deletion, listing, and report tools are not available. Existing agents and reports remain available in the Clifton console.
- **Plugin skill.** `clifton-research` now covers the ask-and-poll flow, direct data through `clifton_search` and `clifton_retrieve`, and Vault downloads.
- **Host testing.** Claude Code over OAuth was tested end to end. Claude Desktop, Claude Web, and ChatGPT Web setups are supported but have not been separately acceptance-tested on v2.

## [0.1.3] - 2026-11-18

- Kept the packaged plugin on the v1 endpoint with its matching ask and Vault-listing instructions while the v2 reference was published ahead of cutover. This reversed the agent-tool instructions added in 0.1.2; agent management remains available in the Clifton console.

## V1 history

- Data Vault listing and authorized API-key agent creation were documented in v1. Delete-agent was also documented before the v2 retirement plan. See the [v1 reference](https://github.com/cliftonai/clifton-mcp/blob/346c08995539fea38a5192835dcb69d9ee4e172c/AGENT-TOOLS.md) for the historical contract.

## [0.1.2] - 2026-06-12

- **Skill covers agents.** The `clifton-research` skill now triggers for agent flows — creating standing monitors (create/watch/alert/schedule) and reading agents and reports — and teaches hosts the creation loop: relay clarifying questions and previews, continue on the same `session_id`, never confirm on the user's behalf, and treat only `status: "enabled"` as created.
- **Broad-condition guidance.** [AGENT-TOOLS.md](AGENT-TOOLS.md) documents host behavior during creation and how to handle conditions like "any company over $500B": pass the condition through in the user's words; if the preview resolves it to a fixed list of names, surface that and let the user choose before confirming — fixed lists do not update by themselves.

## [0.1.1] - 2026-06-10

- **Agent tools.** `clifton_create_agent` (OAuth sign-in required) creates a recurring Clifton agent through a multi-turn conversation, with optional `output_mode: chat | models`. Three read tools — `clifton_list_agents`, `clifton_list_agent_reports`, `clifton_get_agent_report` — list your agents and fetch their reports; available to both OAuth and API-key callers (API-key callers pass `owner_email`). Guide: [AGENT-TOOLS.md](AGENT-TOOLS.md).
- **OAuth sign-in for Claude Code.** The server accepts loopback redirect URIs and Clifton sign-in registers the fixed callback port `8765`; the plugin pins `oauth.callbackPort: 8765` in its config. Manual setup: `claude mcp add --transport http --callback-port 8765 clifton https://ai.cliftonapi.com/v1/mcp`.
- If a connected host doesn't show the new tools, reconnect it — see [TROUBLESHOOTING.md](TROUBLESHOOTING.md) (an app restart alone is not enough on Claude Desktop).

## [0.1.0]

- Initial public docs + Claude Code plugin (`clifton-mcp`).
