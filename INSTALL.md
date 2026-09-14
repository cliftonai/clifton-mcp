# Connect Clifton to your AI app

Use Clifton for markets and finance research, recurring agents, and authorized Data Vault files. You need a [Clifton account](https://console.cliftonapi.com).

Every setup uses this MCP server URL:

```text
https://ai.cliftonapi.com/v1/mcp
```

Choose your app. **OAuth** means signing in with your Clifton account in a browser. For an **API key**, create one under **Settings → API keys** in the Clifton console.

| App | Setup |
| --- | --- |
| [Claude Desktop](#claude-desktop) | Custom connector, OAuth |
| [Claude Web](#claude-web) | Custom connector, OAuth |
| [ChatGPT Desktop](#chatgpt-desktop) | Streamable HTTP, OAuth |
| [ChatGPT Web](#chatgpt-web) | Developer mode, OAuth |

Developer tools: [Claude Code](#claude-code), [Codex](#codex), and [Cursor](#cursor).

## Claude Desktop

1. Open **Customize → Connectors → + → Add custom connector**. Some versions put **Connectors** under **Settings**.
2. Enter **Name:** `Clifton` and **Remote MCP server URL:** `https://ai.cliftonapi.com/v1/mcp`. Leave optional OAuth client fields empty.
3. Add the connector, select **Connect** if prompted, and sign in to Clifton.
4. In a new chat, open **+ → Connectors**, enable Clifton, and [try a question](#check-the-connection).

**Team / Enterprise:** an owner first adds the connector under **Organization settings → Connectors**. Each member then connects their own Clifton account.

[Connector dialog screenshot](assets/screenshots/claude-desktop-add-custom-connector.png) · [Anthropic's setup guide](https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp)

## Claude Web

Open [claude.ai](https://claude.ai) and follow the [Claude Desktop steps](#claude-desktop): **Customize → Connectors → + → Add custom connector**, using the same Clifton name, URL, and sign-in.

If you already connected Clifton in Claude Desktop with the same Claude account, enable it from **+ → Connectors** in your web chat. You do not need to add it again. [Anthropic's desktop and web guide](https://support.claude.com/en/articles/11725091-when-to-use-desktop-and-web-connectors)

## ChatGPT Desktop

1. Open **Settings → MCP servers → Add server** (called **Connect to a custom MCP** in some versions).
2. Select **Streamable HTTP**. Enter **Name:** `clifton` and **URL:** `https://ai.cliftonapi.com/v1/mcp`. Leave API-key headers and optional OAuth client fields empty.
3. Save, select **Authenticate**, and sign in with your Clifton account in the browser.
4. Return to the app, select **Restart** if prompted, and [try a question](#check-the-connection).

**API-key alternative:** add the `X-API-Key` header with your Clifton API key instead of authenticating with OAuth. If your app has no header editor, use the [configuration example](examples/codex.md#api-key).

If your app has no MCP server settings, use [ChatGPT Web](#chatgpt-web). Clifton uses an HTTP URL, so leave STDIO command fields empty. The desktop app and Codex CLI share local MCP configuration; Web setup is separate. [OpenAI's MCP guide](https://learn.chatgpt.com/docs/extend/mcp)

## ChatGPT Web

Create a custom MCP connection on [chatgpt.com](https://chatgpt.com). Developer mode is available on eligible paid accounts, subject to workspace policy.

1. Open **Settings → Security and login** and enable **Developer mode**. If unavailable, check your account eligibility or ask your workspace administrator.
2. Open [Plugins](https://chatgpt.com/plugins) and select **+** to add a developer-mode app.
3. Enter **Name:** `Clifton`, **Description:** `Markets and finance research`, and **MCP server URL:** `https://ai.cliftonapi.com/v1/mcp`.
4. Choose **OAuth**. Leave optional client ID and secret empty; if asked for a registration method, choose **Dynamic Client Registration (DCR)**. Create the connection and complete Clifton sign-in when prompted.
5. In a new chat, choose **+ → Developer mode**, select Clifton, and [try a question](#check-the-connection).

Clifton also offers tools that create or delete agents. Review the selected tools and any action confirmation before proceeding. See [Agent tools](AGENT-TOOLS.md).

If sign-in fails, send [support](SUPPORT.md) the error. [OpenAI's Developer mode guide](https://developers.openai.com/api/docs/guides/developer-mode) · [Connection walkthrough](https://developers.openai.com/plugins/deploy/connect-chatgpt)

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

**Desktop:** use the [ChatGPT Desktop server settings](#chatgpt-desktop).

**CLI:** add the server and sign in:

```bash
codex mcp add clifton --url https://ai.cliftonapi.com/v1/mcp
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

Check that the app actually calls **`clifton_ask`** and returns a sourced answer. A response that merely lists tool names does not prove the connection works.

Available tools depend on your account and authentication method. See [API.md](API.md) for research, agent, and Data Vault tools.

## Manage access and get help

- **Disconnect:** remove Clifton in your app's connector or MCP settings.
- **Rotate a key:** create a replacement in Clifton, update the app, then delete the old key.
- **Trouble connecting:** see [Troubleshooting](TROUBLESHOOTING.md) or [Support](SUPPORT.md). Include your app, the exact error, and the request ID if shown. Never include an API key.
