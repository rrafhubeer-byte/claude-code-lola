# 03 Agent Orchestration

Purpose: support local subagents, spawned workers, agent memory snapshots, and multi-agent coordination.

Approximate source size: very large. Do not load raw as one prompt. Start with the key files, then add dependencies as needed.

## Source Paths

Primary:

- `src/tools/AgentTool/AgentTool.tsx`
- `src/tools/AgentTool/runAgent.ts`
- `src/tools/AgentTool/forkSubagent.ts`
- `src/tools/AgentTool/resumeAgent.ts`
- `src/tools/AgentTool/agentToolUtils.ts`
- `src/tools/AgentTool/agentMemory.ts`
- `src/tools/AgentTool/agentMemorySnapshot.ts`
- `src/tools/AgentTool/builtInAgents.ts`
- `src/tools/AgentTool/built-in/*`
- `src/tasks/LocalAgentTask/LocalAgentTask.tsx`
- `src/tasks/RemoteAgentTask/RemoteAgentTask.tsx`
- `src/tasks/InProcessTeammateTask/*`
- `src/utils/swarm/*`
- `src/coordinator/coordinatorMode.ts`

Secondary:

- `src/utils/agentContext.ts`
- `src/utils/agentId.ts`
- `src/utils/agentSwarmsEnabled.ts`
- `src/utils/standaloneAgent.ts`
- `src/utils/teammate*.ts`
- `src/utils/teamDiscovery.ts`

## Implement

- Agent spawn/resume/stop API.
- Agent isolation contract: separate state, messages, cwd, permissions, memory snapshot.
- Built-in agent profiles.
- Parent-child message passing.
- Result summarization back to parent.
- Multi-agent coordination queue.

## Required Placeholders

- `TODO_SUBSYSTEM(03-agent-orchestration): worker runtime adapter`
- `TODO_SUBSYSTEM(03-agent-orchestration): process isolation or sandbox boundary`
- `TODO_SUBSYSTEM(03-agent-orchestration): agent UI status renderer`
- `TODO_SUBSYSTEM(03-agent-orchestration): remote agent backend`
- `TODO_SUBSYSTEM(03-agent-orchestration): agent permission relay`

## Public Contract

Expose:

- `AgentController`
- `spawnAgent(request)`
- `resumeAgent(id)`
- `sendAgentInput(id, input)`
- `waitForAgent(id)`
- `stopAgent(id)`

Agent implementation must call the runtime session contract from chunk 01, not duplicate session logic.

