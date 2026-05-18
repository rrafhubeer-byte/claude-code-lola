# AI Subsystem Implementation Instructions

Use this folder as the transfer map for implementing the mechanics from `src/` into another app with a limited context window.

The source repo is too large to load as one prompt. Treat each numbered directory as an independent implementation capsule. Load only one capsule at a time, plus the specific source files it lists. Do not ask the model to recreate the whole repository. Ask it to implement the subsystem contract into the target app.

## Operating Rule

For every subsystem, the model must produce:

1. A minimal implementation in the target app.
2. Explicit placeholders for missing integration points.
3. Stable interfaces between subsystems.
4. Notes for any source behavior intentionally deferred.

The model must not silently skip behavior. If something cannot be implemented because the target app is missing infrastructure, it must create a named placeholder using this format:

```ts
// TODO_SUBSYSTEM(<chunk-id>): <what is missing>
// Expected contract: <input/output/events/state>
// Source reference: <source file path>
```

## Chunk Order

Implement in this order:

1. `01-runtime-session`
2. `02-tool-execution`
3. `04-bash-permissions`
4. `05-memory-compaction`
5. `06-mcp-integrations`
6. `03-agent-orchestration`
7. `07-remote-bridge`
8. `08-cli-commands-sdk`
9. `09-security-auth-policy`
10. `10-platform-utilities`

The order matters because later chunks depend on stable contracts from earlier chunks.

## Context Budget Guidance

For a 200k context model, keep each prompt under about 120k raw source tokens. Reserve the rest for target app code, instructions, and model output. Large chunks must be summarized before implementation.

Safe raw chunks:

- `02-tool-execution`
- `05-memory-compaction`
- `08-cli-commands-sdk`
- `09-security-auth-policy`
- `10-platform-utilities`

Large chunks that need selective loading:

- `01-runtime-session`
- `03-agent-orchestration`
- `04-bash-permissions`
- `06-mcp-integrations`
- `07-remote-bridge`

## Required Target-App Placeholders

Create these placeholders even before all chunks are implemented:

```ts
interface AppRuntimeSession {
  id: string;
  cwd: string;
  messages: RuntimeMessage[];
  tools: ToolRegistry;
  permissions: PermissionController;
  memory: MemoryController;
}

interface ToolRegistry {
  register(tool: ToolDefinition): void;
  get(name: string): ToolDefinition | undefined;
  list(): ToolDefinition[];
}

interface PermissionController {
  canRun(request: PermissionRequest): Promise<PermissionDecision>;
}

interface AgentController {
  spawn(request: AgentSpawnRequest): Promise<AgentHandle>;
  resume(id: string): Promise<AgentHandle>;
  stop(id: string): Promise<void>;
}

interface MemoryController {
  search(query: string): Promise<MemoryRecord[]>;
  write(record: MemoryRecord): Promise<void>;
  compact(sessionId: string): Promise<CompactResult>;
}
```

If the target app uses different names, keep an adapter layer rather than mixing subsystem details directly into UI or route handlers.

## Prompt Template

Use this prompt for each chunk:

```text
You are implementing subsystem <chunk-id> into my app.

Read ai-subsystems/<chunk-id>/README.md first.
Use only the source files listed in that README unless you need a directly imported dependency.

Goal:
- Implement the subsystem contract in my app.
- Keep the public interface stable.
- Add TODO_SUBSYSTEM placeholders for missing app infrastructure.
- Do not implement unrelated UI or product features.
- Do not remove existing app behavior unless required by the subsystem contract.

Output:
- Files changed.
- Public interfaces added.
- Placeholders added.
- Source behaviors deferred.
```

## Completion Criteria

A chunk is complete when:

- It has a working internal state model.
- It exposes the interfaces expected by later chunks.
- It has deterministic error handling.
- It includes placeholders for all unresolved dependencies.
- It can be tested independently with mocked neighboring subsystems.

