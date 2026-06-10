# Clifton Agents over MCP

Create and read **Clifton agents** — standing monitors that watch markets, filings, transcripts, and news for a condition you describe, then generate a report when it fires — directly from an MCP host (Claude Code, Claude Desktop, Cursor, or any client connected per [INSTALL.md](INSTALL.md)).

Four tools:

| Tool | What it does |
|---|---|
| `clifton_create_agent` | Multi-turn conversation that defines and enables a new agent |
| `clifton_list_agents` | List your agents (name, status, schedule) |
| `clifton_list_agent_reports` | List reports your agents have generated |
| `clifton_get_agent_report` | Fetch one report's content |

Authentication ([API.md → Authentication](API.md#authentication)) differs by tool:

- **`clifton_create_agent` requires OAuth sign-in.** It is not available to API-key callers — the server omits it from their tool list. Creation acts as a specific user, and an org-scoped API key has no user identity.
- **The three read tools accept either** OAuth or an `X-API-Key` header.

The canonical schemas live at `https://ai.cliftonapi.com/.well-known/mcp-tools.json`.

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

### Choosing the output type: `output_mode`

| Value | The agent produces |
|---|---|
| `chat` | A written research report or alert |
| `models` | A financial model (spreadsheet) it builds or maintains |

Optional. If you omit it, Clifton confirms the output type during the preview. Set it explicitly when the request is unambiguous ("maintain a DCF for NVDA" → `models`).

### Agent limit

Organizations have a cap on concurrently **enabled** agents. If creation would exceed it, the agent is saved with status `disabled` (the console calls this *paused*) instead, and the response is `status: "error"` with a `workflow_limit_exceeded` block carrying your `limit` and current `active_count`. Pause or delete an agent in the [console](https://console.cliftonapi.com), then resume the new one — nothing is lost.

### Timeouts

Creation turns are slower than `clifton_ask` — the commit turn generates and validates the agent and can take a couple of minutes. If your host enforces a shorter tool timeout and a turn is cut off, call again with the same `session_id`; the conversation resumes where it left off.

### Attachments

File attachments from the MCP host are **not** supported. The `attachments` field accepts Clifton Data Vault document ids only (files already uploaded via the console). To ground an agent in specific data, reference it in the message or upload the file to your Data Vault first.

---

## Reading agents and reports

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
