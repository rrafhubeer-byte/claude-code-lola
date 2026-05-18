# 02 Tool Execution

Purpose: define and execute tools through a stable registry, including streaming output, validation, and result storage.

Approximate source size: medium, safe to load in one prompt with target app code.

## Source Paths

- `src/Tool.ts`
- `src/tools.ts`
- `src/tools/utils.ts`
- `src/services/tools/StreamingToolExecutor.ts`
- `src/services/tools/toolExecution.ts`
- `src/services/tools/toolHooks.ts`
- `src/services/tools/toolOrchestration.ts`
- `src/utils/toolPool.ts`
- `src/utils/toolErrors.ts`
- `src/utils/toolResultStorage.ts`
- `src/utils/toolSchemaCache.ts`
- `src/tools/FileReadTool/*`
- `src/tools/FileWriteTool/*`
- `src/tools/FileEditTool/*`
- `src/tools/GlobTool/*`
- `src/tools/GrepTool/*`
- `src/tools/TodoWriteTool/*`
- `src/tools/SyntheticOutputTool/*`

## Implement

- Tool definition schema.
- Tool registry.
- Tool call validation.
- Streaming and non-streaming execution.
- Tool result normalization.
- Tool error classification.
- Result persistence for large outputs.

## Required Placeholders

- `TODO_SUBSYSTEM(02-tool-execution): target app filesystem policy`
- `TODO_SUBSYSTEM(02-tool-execution): UI renderer for tool progress`
- `TODO_SUBSYSTEM(02-tool-execution): large result blob storage`
- `TODO_SUBSYSTEM(02-tool-execution): hook dispatch adapter`

## Public Contract

Expose:

- `ToolDefinition`
- `ToolCall`
- `ToolResult`
- `ToolRegistry`
- `executeTool(call, context)`
- `streamToolExecution(call, context)`

