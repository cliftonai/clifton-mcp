# Changelog

Public changes to the Clifton MCP tool surface.

## [0.2.0] - Unreleased

The existing MCP stays running until all v2 tools are fully tested and the cutover is separately approved. V2 includes v1 retirement and agent-tool removal. Merging or releasing the v2 Router branch, merging or publishing this plugin, or redirecting the old endpoint requires that cutover approval. This also applies to another Router release containing these changes.

- **MCP v2 connection.** The plugin targets `https://ai.cliftonapi.com/v2/mcp`. After the approved cutover, existing manual connections update their URL and reconnect to refresh the tool list. The old endpoint returns upgrade instructions after its separately approved retirement.
- **Long-running answers.** `clifton_ask` returns saved session/message identifiers. `clifton_get_response` reads the current saved state and returns as soon as storage completes. Callers use their own timing facility to wait `poll_after_seconds` (currently ten seconds) after acknowledgement and between running reads. Polling never starts another answer or adds another answer charge.
- **Research continuation.** When available for the caller, `effort: research` supports saved questions and pauses. Stop polling when input or a resume is required.
- **Persistence required.** MCP v2 omits incognito. A lost initial acknowledgement must not trigger automatic resubmission. See [API.md](API.md).


- **Data Vault downloads.** Use `clifton_list_vault_files` to find an authorized file ID, then `clifton_get_vault_file` for a signed S3 download URL valid for up to 15 minutes. Use `clifton_ask.attachments` when the request is to analyze the file instead.
- **Vault owner permissions.** OAuth defaults to the signed-in user; selecting another owner requires `vault:delegate`, not ordinary manager status or workflow delegation. API-key calls require an authorized owner selector. Private downloads remain inside the authenticated organization even for administrators; existing GLOBAL-file access remains.
- **Six-tool scope.** V2 exposes ask, get response, search, retrieve, list Vault files, and get Vault file. Agent creation, deletion, listing, and report tools are unavailable in v2. Existing agents and reports remain available in the Clifton console.

## [0.1.2] - 2026-06-12

- **Skill covers agents.** The `clifton-research` skill now triggers for agent flows — creating standing monitors (create/watch/alert/schedule) and reading agents and reports — and teaches hosts the creation loop: relay clarifying questions and previews, continue on the same `session_id`, never confirm on the user's behalf, and treat only `status: "enabled"` as created.
- **Broad-condition guidance.** [AGENT-TOOLS.md](AGENT-TOOLS.md) documents host behavior during creation and how to handle conditions like "any company over $500B": pass the condition through in the user's words; if the preview resolves it to a fixed list of names, surface that and let the user choose before confirming — fixed lists do not update by themselves.

## [0.1.1] - 2026-06-10

- **Agent tools.** `clifton_create_agent` (OAuth sign-in required) creates a recurring Clifton agent through a multi-turn conversation, with optional `output_mode: chat | models`. Three read tools — `clifton_list_agents`, `clifton_list_agent_reports`, `clifton_get_agent_report` — list your agents and fetch their reports; available to both OAuth and API-key callers (API-key callers pass `owner_email`). Guide: [AGENT-TOOLS.md](AGENT-TOOLS.md).
- **OAuth sign-in for Claude Code.** The server accepts loopback redirect URIs and Clifton sign-in registers the fixed callback port `8765`; the plugin pins `oauth.callbackPort: 8765` in its config. Manual setup: `claude mcp add --transport http --callback-port 8765 clifton https://ai.cliftonapi.com/v1/mcp`.
- If a connected host doesn't show the new tools, reconnect it — see [TROUBLESHOOTING.md](TROUBLESHOOTING.md) (an app restart alone is not enough on Claude Desktop).

## [0.1.0]

- Initial public docs + Claude Code plugin (`clifton-mcp`).
