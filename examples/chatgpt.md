# ChatGPT Web — preview connection settings

V2 host acceptance is pending. Follow the current [OpenAI connection guide](https://help.openai.com/en/articles/12584461-connecting-custom-mcp-servers-in-chatgpt) and your workspace policy. In the current Plugins flow, use **+ → Add custom MCP server**; enabling Developer Mode first is not required.

- **Name:** `Clifton`
- **MCP server URL after cutover:** `https://ai.cliftonapi.com/v2/mcp`
- **Authentication:** OAuth. Leave optional client credentials empty when Dynamic Client Registration is offered.

Review the requested access before creating a connection. Do not replace your live v1 connection while v2 is a preview.

Full setup: [ChatGPT Web](../INSTALL.md#chatgpt-web). ChatGPT Desktop has not been verified; it does not share Codex's configuration by assumption.
