# 01 Runtime Session

Purpose: provide the central conversation/session engine that every other subsystem plugs into.

Approximate source size: large, load selectively.

## Source Paths

- `src/query.ts`
- `src/QueryEngine.ts`
- `src/context.ts`
- `src/history.ts`
- `src/assistant/sessionHistory.ts`
- `src/utils/messages.ts`
- `src/utils/messageQueueManager.ts`
- `src/utils/messagePredicates.ts`
- `src/utils/sessionStorage.ts`
- `src/utils/sessionStoragePortable.ts`
- `src/utils/sessionState.ts`
- `src/utils/sessionRestore.ts`
- `src/utils/sessionStart.ts`
- `src/utils/sessionActivity.ts`
- `src/utils/sessionEnvironment.ts`
- `src/utils/sessionEnvVars.ts`
- `src/utils/sessionTitle.ts`
- `src/utils/sessionUrl.ts`
- `src/utils/queryContext.ts`
- `src/utils/queryHelpers.ts`
- `src/utils/QueryGuard.ts`

## Implement

- Session lifecycle: create, resume, persist, restore, stop.
- Message model: user, assistant, tool use, tool result, system, compacted summary.
- Event queue between model calls, tools, and UI.
- Runtime context builder that gathers cwd, model settings, messages, tools, memory, and permissions.
- Guardrails for concurrent queries and cancellation.

## Required Placeholders

- `TODO_SUBSYSTEM(01-runtime-session): model provider call boundary`
- `TODO_SUBSYSTEM(01-runtime-session): target app persistence adapter`
- `TODO_SUBSYSTEM(01-runtime-session): target app UI event sink`
- `TODO_SUBSYSTEM(01-runtime-session): telemetry/log sink`

## Public Contract

Expose:

- `createRuntimeSession(input)`
- `resumeRuntimeSession(id)`
- `appendRuntimeMessage(sessionId, message)`
- `runRuntimeTurn(sessionId, input)`
- `cancelRuntimeTurn(sessionId)`

Do not expose raw source repo storage internals to other chunks.

