# ChatGPT Web — connection settings

Upgrading from v1? Change an existing `/v1/mcp` URL to `/v2/mcp` and reconnect instead of adding a second connection.

This setup is supported but has not been separately acceptance-tested on v2. Follow the current [OpenAI connection guide](https://help.openai.com/en/articles/12584461-connecting-custom-mcp-servers-in-chatgpt) and your workspace policy. In the current Plugins flow, use **+ → Add custom MCP server**; enabling Developer Mode first is not required.

- **Name:** `Clifton`
- **MCP server URL:** `https://ai.cliftonapi.com/v2/mcp`
- **Authentication:** OAuth. Leave optional client credentials empty when Dynamic Client Registration is offered.

Review the requested access before creating a connection.

Full setup: [ChatGPT Web](../INSTALL.md#chatgpt-web). ChatGPT Desktop has not been verified; it does not share Codex's configuration by assumption.
