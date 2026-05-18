# 06 MCP Integrations

Purpose: support Model Context Protocol servers, MCP tools, auth, resources, and connection lifecycle.

Approximate source size: large. Load selectively.

## Source Paths

- `src/services/mcp/*`
- `src/tools/MCPTool/*`
- `src/tools/McpAuthTool/*`
- `src/tools/ListMcpResourcesTool/*`
- `src/tools/ReadMcpResourceTool/*`
- `src/entrypoints/mcp.ts`
- `src/utils/mcp/*`
- `src/utils/mcpValidation.ts`
- `src/utils/mcpOutputStorage.ts`
- `src/utils/mcpWebSocketTransport.ts`
- `src/types/plugin.ts`
- `src/plugins/*`
- `src/services/plugins/*`

## Implement

- MCP server config model.
- Connection manager.
- Tool discovery and normalization.
- Resource listing/reading.
- MCP auth flow boundary.
- Tool invocation adapter into chunk 02.
- MCP output storage.

## Required Placeholders

- `TODO_SUBSYSTEM(06-mcp-integrations): OAuth browser/device flow`
- `TODO_SUBSYSTEM(06-mcp-integrations): secure token storage`
- `TODO_SUBSYSTEM(06-mcp-integrations): remote MCP transport`
- `TODO_SUBSYSTEM(06-mcp-integrations): plugin marketplace integration`
- `TODO_SUBSYSTEM(06-mcp-integrations): MCP UI management surface`

## Public Contract

Expose:

- `McpConnectionManager`
- `connectMcpServer(config)`
- `listMcpTools(serverId)`
- `invokeMcpTool(call)`
- `listMcpResources(serverId)`
- `readMcpResource(resourceId)`

