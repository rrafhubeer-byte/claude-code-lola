# 05 Memory And Compaction

Purpose: support persistent memories, session memory, summaries, and context compaction.

Approximate source size: medium, safe to load in one or two prompts.

## Source Paths

Memory:

- `src/memdir/*`
- `src/services/SessionMemory/*`
- `src/services/extractMemories/*`
- `src/services/teamMemorySync/*`
- `src/utils/memory/*`
- `src/utils/teamMemoryOps.ts`
- `src/utils/memoryFileDetection.ts`

Compaction:

- `src/services/compact/*`
- `src/commands/compact/*`
- `src/utils/tokenBudget.ts`
- `src/query/tokenBudget.ts`
- `src/services/tokenEstimation.ts`

## Implement

- Memory record schema.
- Local memory storage adapter.
- Memory search/ranking.
- Session memory extraction.
- Compact summary generation.
- Token budget estimation.
- Post-compact cleanup hook.

## Required Placeholders

- `TODO_SUBSYSTEM(05-memory-compaction): embedding or semantic search provider`
- `TODO_SUBSYSTEM(05-memory-compaction): model summarization provider`
- `TODO_SUBSYSTEM(05-memory-compaction): target app memory storage`
- `TODO_SUBSYSTEM(05-memory-compaction): team/shared memory sync backend`

## Public Contract

Expose:

- `MemoryController`
- `extractMemories(messages)`
- `findRelevantMemories(query)`
- `compactSession(sessionId)`
- `estimateTokenBudget(input)`

