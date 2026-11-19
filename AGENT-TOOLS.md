# Agent tools and MCP v2

MCP v2 does not expose agent creation, deletion, listing, or report tools. Use the [Clifton console](https://console.cliftonapi.com) to manage recurring agents and read their reports. Existing agents, schedules, and reports are retained.

The existing MCP stays running until all v2 tools are fully tested and the cutover is separately approved. This notice describes v2; it does not retire the old live connection.

V2 supports research, search/retrieve, and Data Vault listing/downloads. See the [tool reference](API.md) and [connection guide](INSTALL.md).

The [pinned historical v1 agent guide](https://github.com/cliftonai/clifton-mcp/blob/346c08995539fea38a5192835dcb69d9ee4e172c/AGENT-TOOLS.md) records the earlier contract. The plugin skill directs requests to the console and forbids invoking these tools, even if the v1 server still lists them.
