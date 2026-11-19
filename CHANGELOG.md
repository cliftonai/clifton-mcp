# Changelog

Public changes to the Clifton MCP tool surface.

## [0.1.3] - 2026-11-18

- Keep the packaged plugin on the live v1 endpoint with its matching ask and Vault-listing instructions while publishing the v2 preview reference. Agent management remains available in the Clifton console.
- Browser/Desktop v2 host acceptance remains pending; preview instructions do not establish support.

## [0.2.0] - Unreleased

The existing MCP stays running until all v2 tools are fully tested and the cutover is separately approved. V2 includes v1 retirement and agent-tool removal. The packaged plugin remains on v1 until the live cutover; the preview reference does not move the live endpoint.

- **MCP v2 connection.** The plugin targets `https://ai.cliftonapi.com/v2/mcp`. After the approved cutover, existing manual connections update their URL and reconnect to refresh the tool list. The old endpoint returns upgrade instructions after its separately approved retirement.
- **Long-running answers.** `clifton_ask` returns saved session/message identifiers. `clifton_get_response` reads the current saved state and returns as soon as storage completes. Callers use their own timing facility to wait `poll_after_seconds` (currently ten seconds) after acknowledgement and between running reads. Polling never starts another answer or adds another answer charge.
- **Model and effort.** `model` chooses `instant` or `deep-thinking`; `effort` chooses `extra_low`, `low`, `medium`, `high`, `extra_high`, or `ultra`. Defaults are Instant/medium. V2 rejects the former profile values in `effort` and does not accept `model: research`. Deep Thinking retains its existing OAuth access requirement; API keys use Instant with any standard depth.
- **Deep Thinking continuation.** Acknowledgements and recognized saved responses return both choices. Resume a saved question or pause with the exact model/effort pair, returned IDs, and original `response_format` if supplied. Omission applies defaults anew, not inheritance. Stop polling when action is required.
- **Persistence required.** MCP v2 omits incognito. A lost initial acknowledgement must not trigger automatic resubmission. See [API.md](API.md).

- **Data Vault downloads.** Use `clifton_list_vault_files` to find an authorized file ID, then `clifton_get_vault_file` for a signed S3 download URL valid for up to 15 minutes. Use `clifton_ask.attachments` when the request is to analyze the file instead.
- **Vault owner permissions.** OAuth defaults to the signed-in user; selecting another owner requires `vault:delegate`, not ordinary manager status or workflow delegation. API-key calls require an authorized owner selector. Private downloads remain inside the authenticated organization even for administrators; existing GLOBAL-file access remains.
- **Six-tool scope.** V2 exposes ask, get response, search, retrieve, list Vault files, and get Vault file. Agent creation, deletion, listing, and report tools are unavailable in v2. Existing agents and reports remain available in the Clifton console.

## V1 history retained before the v2 preview

- Data Vault listing and authorized API-key agent creation were documented before this preview. Delete-agent was also documented before the v2 retirement plan. See the [v1 reference](https://github.com/cliftonai/clifton-mcp/blob/346c08995539fea38a5192835dcb69d9ee4e172c/AGENT-TOOLS.md) for the historical contract.

## [0.1.2] - 2026-06-12

- **Skill covers agents.** The `clifton-research` skill now triggers for agent flows — creating standing monitors (create/watch/alert/schedule) and reading agents and reports — and teaches hosts the creation loop: relay clarifying questions and previews, continue on the same `session_id`, never confirm on the user's behalf, and treat only `status: "enabled"` as created.
- **Broad-condition guidance.** [AGENT-TOOLS.md](AGENT-TOOLS.md) documents host behavior during creation and how to handle conditions like "any company over $500B": pass the condition through in the user's words; if the preview resolves it to a fixed list of names, surface that and let the user choose before confirming — fixed lists do not update by themselves.

## [0.1.1] - 2026-06-10

- **Agent tools.** `clifton_create_agent` (OAuth sign-in required) creates a recurring Clifton agent through a multi-turn conversation, with optional `output_mode: chat | models`. Three read tools — `clifton_list_agents`, `clifton_list_agent_reports`, `clifton_get_agent_report` — list your agents and fetch their reports; available to both OAuth and API-key callers (API-key callers pass `owner_email`). Guide: [AGENT-TOOLS.md](AGENT-TOOLS.md).
- **OAuth sign-in for Claude Code.** The server accepts loopback redirect URIs and Clifton sign-in registers the fixed callback port `8765`; the plugin pins `oauth.callbackPort: 8765` in its config. Manual setup: `claude mcp add --transport http --callback-port 8765 clifton https://ai.cliftonapi.com/v1/mcp`.
- If a connected host doesn't show the new tools, reconnect it — see [TROUBLESHOOTING.md](TROUBLESHOOTING.md) (an app restart alone is not enough on Claude Desktop).

## [0.1.0]

- Initial public docs + Claude Code plugin (`clifton-mcp`).
