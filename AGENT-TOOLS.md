# Clifton Agents over MCP

Use an MCP host to create, delete, and read **Clifton agents**. These standing monitors watch markets, filings, transcripts, and news, then generate a report when a condition fires. Supported hosts include Claude Code, Claude Desktop, Cursor, and any client connected per [INSTALL.md](INSTALL.md).

Five tools:

| Tool | What it does |
|---|---|
| `clifton_create_agent` | Multi-turn conversation that defines and enables a new agent |
| `clifton_delete_agent` | Delete one agent owned by the signed-in user |
| `clifton_list_agents` | List your agents (name, status, schedule) |
| `clifton_list_agent_reports` | List reports your agents have generated |
| `clifton_get_agent_report` | Fetch one report's content |

Authentication ([API.md → Authentication](API.md#authentication)) differs by tool:

- **`clifton_create_agent` accepts OAuth and approved API-key callers.** OAuth callers default to the signed-in user. API-key callers see the tool only when Clifton has enabled API-key creation for their organization; they must pass an `owner_email` for a manager in that organization.
- **`clifton_delete_agent` requires OAuth sign-in.** The server omits it from API-key tool lists because deletion acts as the signed-in user.
- **The three read tools accept either** OAuth or an `X-API-Key` header.

The public schemas are listed at `https://ai.cliftonapi.com/.well-known/mcp-tools.json`. Authenticated `tools/list` applies user and organization gates and is authoritative for the caller. If your host does not show these tools, reconnect the Clifton connection. See [TROUBLESHOOTING.md](TROUBLESHOOTING.md#your-host-shows-fewer-or-older-clifton-tools-than-expected-or-rejects-a-tool-argument).

---

## Creating an agent

`clifton_create_agent` is conversational. Describe what you want watched; Clifton asks clarifying questions, shows a preview, and commits the agent when you confirm.

```text
You: Create an agent that alerts me when any S&P 500 bank announces a CEO change.
Clifton: (clarifying question) Should this cover interim appointments, or only permanent successions?
You: Only permanent.
Clifton: (preview) … Confirm to enable.
You: Confirm.
Clifton: Agent enabled — view it at <agent_url>.
```

Each response carries a `status`:

| `status` | Meaning |
|---|---|
| `creating` | Clifton asked a clarifying question or showed a preview — reply on the same `session_id` to continue |
| `enabled` | The agent is committed and live; `workflow_id` and `agent_url` are set |
| `error` | Preflight, quota, or agent-limit failure; `error_message` explains |

Pass `session_id` from each response into your next call. Hosts do this automatically when you just keep talking.

### Host behavior during creation

For MCP hosts (and host-side skills) driving this tool on a user's behalf:

- Relay Clifton's clarifying questions and previews to the user **verbatim**; do not answer them or confirm for the user.
- `creating` means *continue the same conversation* — reply on the same `session_id`, not with a fresh call.
- Treat the agent as created only on `status: "enabled"` with an `agent_url` or `workflow_id`.

### Broad or changing conditions

Some requests define the watched set by **condition** rather than by name — "any company over $500B", "banks above a deposit threshold", "stocks in the retail sector". Pass the condition through in the user's words. Clifton's preview may resolve such a condition into a fixed list of names as of today; the preview will say so. If the user intended ongoing coverage (companies that meet the condition in the future), surface that difference and ask whether they want a fixed snapshot or ongoing coverage — a snapshot will not update by itself.

### Choosing the output type: `output_mode`

| Value | The agent produces |
|---|---|
| `chat` | A written research report or alert |
| `models` | A financial model (spreadsheet) it builds or maintains |

Optional. If you omit it, Clifton confirms the output type during the preview. Set it explicitly when the request is unambiguous ("maintain a DCF for NVDA" → `models`).

### Agent limit

Organizations have a cap on concurrently **enabled** agents. If creation would exceed it, the agent is saved with status `disabled` (the console calls this *paused*) instead, and the response is `status: "error"` with a `workflow_limit_exceeded` block carrying your `limit` and current `active_count`. Pause an agent in the [console](https://console.cliftonapi.com) or delete one with `clifton_delete_agent`, then resume the new one.

### Timeouts

Creation turns are slower than `clifton_ask` — the commit turn generates and validates the agent and can take a couple of minutes. If your host enforces a shorter tool timeout and a turn is cut off, call again with the same `session_id`; the conversation resumes where it left off.

### Attachments

File attachments from the MCP host are **not** supported. The `attachments` field accepts Clifton Data Vault file IDs only: files already uploaded via the console. Use `clifton_list_vault_files` to find IDs for files the signed-in user can access, then pass those IDs to `clifton_create_agent.attachments` for agent creation or `clifton_ask.attachments` for one-off analysis.

---

## Deleting an agent

`clifton_delete_agent` accepts one `agent_id`. Use `clifton_list_agents` first when the user names an agent but does not provide its id.

- Call deletion only after the user clearly asks to delete or remove the agent.
- If several agents match the request, show the matches and ask the user to choose. Do not guess.
- Do not interpret requests to pause an agent or stop notifications as deletion.
- Deletion stops future runs and cannot be undone. Existing reports remain available through the report tools.

The tool returns the deleted `agent_id`, `status: "deleted"`, and `reports_retained: true`.

---

## Reading agents and reports

Read calls count toward your organization's MCP rate limits (the same per-org caps as other Clifton MCP tools); they do not consume chat quota.

### Who you read as: `owner_email`

- **OAuth callers** (signed in): `owner_email` is optional and defaults to you. You may pass another member of your organization to read their agents.
- **API-key callers** (org-scoped key, no user identity): `owner_email` is required and must belong to the key's organization.

Cross-organization reads are rejected.

### `clifton_list_agents`

Returns your agents, newest first, with their `status` — one of `enabled`, `disabled` (shown as *paused* in the console), `creating`, `error`, or `awaiting_user_confirmation` — plus prompts, recipients, and report counts. Filter with `status` or `tags`. Paginated with `limit` (default 10, max 1000) and `offset`.

### `clifton_list_agent_reports`

Returns report summaries, newest first: agent, fired time, outcome, and console link. Filter to one agent with `agent_id`. Same `limit`/`offset` pagination.

### `clifton_get_agent_report`

Fetches one report by id. The shape depends on the agent's output type and the run's outcome:

| Report | You get |
|---|---|
| Generated, `chat` | The full report as markdown, plus a console link |
| Generated, `models` | A time-limited download link for the spreadsheet, plus a console link |
| Not generated (rate-limited / validation-failed) | A `reason` explaining why no report was produced |

Every report includes a supporting-evidence summary: what the agent matched, with source links and match confidence.

---

## See also

- [API.md](API.md) — endpoint, authentication, full tool reference
- [INSTALL.md](INSTALL.md) — per-host setup, including the Claude Code OAuth callback port
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) — common failures
