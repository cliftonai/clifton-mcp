# Connect Clifton to your AI app

Use Clifton for markets and finance research and authorized Data Vault file discovery, downloads, and uploads. You need a [Clifton account](https://console.cliftonapi.com).

Every setup uses this MCP server URL:

```text
https://ai.cliftonapi.com/v2/mcp
```

**Upgrading from v1:** `/v1/mcp` is retired and only returns an upgrade notice. Change the existing connection's URL from `/v1/mcp` to `/v2/mcp`, then reconnect and refresh its tools. For the Claude Code plugin, update `clifton-mcp` to 0.2.0 and restart or reconnect it. Do not add a second Clifton connection. V2 supports research, search/retrieve, and Vault listing, downloads, and uploads; agent tools are unavailable in v2.

**Testing status:** Claude Code over OAuth has been tested end to end on v2. The other setups below, including Claude Desktop, Claude Web, and ChatGPT Web, are supported but have not been separately acceptance-tested on v2.

Choose your app. **OAuth** means signing in with your Clifton account in a browser. For an **API key**, create one under **Settings → API keys** in the Clifton console.

| App | Setup |
| --- | --- |
| [Claude Desktop](#claude-desktop) | Custom connector, OAuth |
| [Claude Web](#claude-web) | Custom connector, OAuth |
| [ChatGPT Desktop](#chatgpt-desktop) | Not verified; use web setup |
| [ChatGPT Web](#chatgpt-web) | Custom plugin, OAuth; workspace policy applies |

Developer tools: [Claude Code](#claude-code), [Codex](#codex), and [Cursor](#cursor).

## Claude Desktop

1. Open **Customize → Connectors → + → Add custom connector**. Some versions put **Connectors** under **Settings**.
2. Enter **Name:** `Clifton` and **Remote MCP server URL:** `https://ai.cliftonapi.com/v2/mcp`. Leave optional OAuth client fields empty.
3. Add the connector, select **Connect** if prompted, and sign in to Clifton.
4. In a new chat, open **+ → Connectors**, enable Clifton, and [try a question](#check-the-connection).

**Team / Enterprise:** an owner first adds the connector under **Organization settings → Connectors**. Each member then connects their own Clifton account.

[Connector dialog screenshot](assets/screenshots/claude-desktop-add-custom-connector.png) · [Anthropic's setup guide](https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp)

## Claude Web

Open [claude.ai](https://claude.ai) and follow the [Claude Desktop steps](#claude-desktop): **Customize → Connectors → + → Add custom connector**, using the same Clifton name, URL, and sign-in.

If you already connected Clifton in Claude Desktop with the same Claude account, enable it from **+ → Connectors** in your web chat. You do not need to add it again. [Anthropic's desktop and web guide](https://support.claude.com/en/articles/11725091-when-to-use-desktop-and-web-connectors)

## ChatGPT Desktop

Clifton has not verified a custom-MCP setup path in ChatGPT Desktop. Use [ChatGPT Web](#chatgpt-web); do not assume the ChatGPT app shares Codex's MCP configuration.

## ChatGPT Web

An eligible account and workspace permission are required. The current [OpenAI connection guide](https://help.openai.com/en/articles/12584461-connecting-custom-mcp-servers-in-chatgpt) describes web setup and plan-specific permissions.

1. Open [Plugins](https://chatgpt.com/plugins), select **+**, then **Add custom MCP server**. The current Plugins flow does not require enabling Developer Mode first; older interfaces may differ.
2. Enter **Name:** `Clifton` and **MCP server URL:** `https://ai.cliftonapi.com/v2/mcp`. Choose **OAuth**; leave optional client credentials empty when Dynamic Client Registration is offered.
3. Review the requested access and the app's risk warning, then create the plugin and complete Clifton sign-in. Your account or workspace may require an administrator to do this.
4. Review the discovered tools. In a new conversation, select the plugin with **@** and [check answer delivery](#check-the-connection).

MCP v2 has no agent creation or deletion tools. Manage agents in the [Clifton console](https://console.cliftonapi.com).

If sign-in fails, send [support](SUPPORT.md) the error. Never send passwords or tokens.

## Developer tools

### Claude Code

Run these commands inside Claude Code:

```text
/plugin marketplace add cliftonai/clifton-mcp
/plugin install clifton-mcp@clifton-mcp
```

Restart Claude Code, then use `/mcp` to authenticate Clifton. The plugin includes the `clifton-research` skill, which guides Claude Code to use Clifton for finance questions.

If authentication fails, follow [OAuth troubleshooting](TROUBLESHOOTING.md#oauth-sign-in-does-not-complete).

For manual OAuth or API-key setup, see [Claude Code examples](examples/claude-code.md).

### Codex

**Desktop:** Codex is a separate product from ChatGPT. Use the MCP configuration supported by your Codex version; do not assume ChatGPT Desktop settings apply.

**CLI:** add the server and sign in:

```bash
codex mcp add clifton --url https://ai.cliftonapi.com/v2/mcp
codex mcp login clifton
```

Complete Clifton sign-in in the browser. Restart Codex, use `/mcp` to check the connection, then [try a question](#check-the-connection). If Clifton is already configured, update its existing entry instead of adding a duplicate.

For manual configuration or an API key, see the [Codex examples](examples/codex.md). Current versions need no `rmcp_client` feature flag.

**Web / cloud:** [Codex cloud](https://learn.chatgpt.com/docs/cloud) uses separate repository environments. For browser-based research, follow [ChatGPT Web](#chatgpt-web).

### Cursor

Add Clifton's URL and API-key header to `~/.cursor/mcp.json` using the [Cursor example](examples/cursor.md). Restart Cursor, then [try a question](#check-the-connection).

## Check the connection

Start a new chat and ask:

> Use Clifton to find NVIDIA's most recent reported quarterly revenue and cite the source.

Confirm that **`clifton_ask`** returns server-issued IDs, the host waits `poll_after_seconds`, and **`clifton_get_response`** with those exact IDs eventually returns a completed sourced answer. Stop on failures or action-needed states; do not automatically resubmit an uncertain ask. A receipt, tool-name list, or Connected badge alone does not prove answer delivery.

Available tools depend on your account and authentication method. See [API.md](API.md) for research, search/retrieve, and Data Vault tools.

To download a Vault file, ask the app to find it with `clifton_list_vault_files`, then call `clifton_get_vault_file` with its returned `file_id`. The result includes a temporary signed S3 URL. Open that URL or let the app fetch it; a returned URL alone does not mean the app has saved the file. [Vault reference](API.md#clifton_get_vault_file-read-only).

To analyze a local file, a host that can make HTTP PUT requests (Claude Code or Codex) can upload it with `clifton_upload_vault_file`: start, PUT the bytes, then finalize, and pass the returned ID to `clifton_ask.attachments` with Deep Thinking. In Claude Web, Claude Desktop, or ChatGPT, upload the file in the [Clifton console](https://console.cliftonapi.com) instead. [Upload reference](API.md#clifton_upload_vault_file-write).

## Check direct public data

With an eligible organization account, explicitly ask for raw records to use
`clifton_search`, or for ranked documents to use `clifton_retrieve`. Keep
ordinary research questions on `clifton_ask`. Refresh the connection's tool
list if a newly enabled tool is missing; availability depends on the server's
rollout and account access.

For either direct tool, first call with `{"request_json":"{}"}`. Use a scope
returned by the endpoint in a scope-only request to discover its schemas,
then apply the selected tool's `request_json` instructions to build its query.
Confirm the complete body, native
status and discovery flag. `422` with `discovery:true` is schema education;
query data requires 2xx and no discovery flag. Tool names or final prose alone
do not prove a successful read.

Large successful output may appear in a complete host file. Read that file
instead of accepting a clipped preview; if unavailable, request a smaller page
using the returned schema's controls. The 1 MiB server response limit does not
override smaller host limits. A complete client log does not prove the model
received the complete body. [Direct data reference](API.md#direct-public-data).

## Manage access and get help

- **Disconnect:** remove Clifton in your app's connector or MCP settings.
- **Rotate a key:** create a replacement in Clifton, update the app, then delete the old key.
- **Trouble connecting:** see [Troubleshooting](TROUBLESHOOTING.md) or [Support](SUPPORT.md). Include your app, the exact error, and the request ID if shown. Never include an API key.
