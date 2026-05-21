# CODEBASE NAVIGATOR — claude-code-lola
> **PURPOSE**: This file is the AI-agent navigation guide for this 3.2M-token codebase.
> Instead of loading the entire repo, load only this file + the specific sections/files
> relevant to your current task. Total size of this navigator: ~50K tokens vs 3.2M raw.

---

## TABLE OF CONTENTS

1. [Repo Overview](#1-repo-overview)
2. [Architecture Layers](#2-architecture-layers)
3. [Subsystem Capsules (ai-subsystems/)](#3-subsystem-capsules)
4. [Execution Flow: Request to Response](#4-execution-flow)
5. [Core Files Quick Reference](#5-core-files-quick-reference)
6. [All Tools — Feature & File Map](#6-all-tools)
7. [All Commands — Feature & File Map](#7-all-commands)
8. [Services Directory — Feature & File Map](#8-services)
9. [Utils Directory — Feature & File Map](#9-utils)
10. [Types & Schemas](#10-types--schemas)
11. [Feature Flags (bun:bundle features)](#11-feature-flags)
12. [Key Dependency Graph](#12-key-dependency-graph)
13. [Web Platform Migration Guide](#13-web-platform-migration-guide)

---

## 1. REPO OVERVIEW

**What it is**: A production-grade agentic AI coding assistant CLI built with TypeScript/Bun.
- **Runtime**: Bun (not Node.js) — uses `bun:bundle` for dead-code-elimination feature flags
- **Total source files**: ~610 TypeScript + 18 TSX
- **Utils alone**: 138 files in `src/utils/`
- **Tools**: 40 tool directories in `src/tools/`
- **Primary AI SDK**: `@anthropic-ai/sdk` (Claude, Bedrock, Vertex AI)

**Root directories**:
```
src/                     — All application source code
ai-subsystems/           — Pre-chunked implementation capsules for AI agents
  INSTRUCTIONS.md        — Operating rules for using these capsules
  01-runtime-session/    — Session + message engine chunk
  02-tool-execution/     — Tool registry + execution chunk
  03-agent-orchestration/— Multi-agent spawning + coordination chunk
  04-bash-permissions/   — Shell + permission classification chunk
  05-memory-compaction/  — Memory + context compaction chunk
  06-mcp-integrations/   — MCP server integration chunk
  07-remote-bridge/      — Remote/bridge/WebSocket transport chunk
  08-cli-commands-sdk/   — CLI commands + SDK entrypoints chunk
  09-security-auth-policy/ — Auth + policy + security chunk
  10-platform-utilities/ — Shared utilities chunk
```

**IMPORTANT**: The `ai-subsystems/` folder already has pre-made implementation capsules with:
- Exact source file lists for each chunk
- Required public interfaces
- Required placeholder comments
- Implementation order (01 → 10 with 04 before 03)

---

## 2. ARCHITECTURE LAYERS

```
┌─────────────────────────────────────────────────────────────┐
│  ENTRYPOINTS (CLI, SDK, MCP, Bridge)                        │
│  src/entrypoints/init.ts  src/entrypoints/sdk/              │
│  src/cli/  src/bridge/                                       │
├─────────────────────────────────────────────────────────────┤
│  QUERY ENGINE (main conversation loop)                       │
│  src/QueryEngine.ts  src/query.ts                           │
├─────────────────────────────────────────────────────────────┤
│  TOOL ORCHESTRATION                                          │
│  src/services/tools/  src/Tool.ts  src/tools.ts             │
├──────────────────────────┬──────────────────────────────────┤
│  TOOLS (40 tools)        │  SERVICES                        │
│  src/tools/              │  src/services/api/               │
│                          │  src/services/mcp/               │
│                          │  src/services/compact/           │
│                          │  src/services/oauth/             │
├──────────────────────────┴──────────────────────────────────┤
│  UTILS (138 files) — shared low-level helpers               │
│  src/utils/                                                  │
├─────────────────────────────────────────────────────────────┤
│  BOOTSTRAP STATE (global singleton)                          │
│  src/bootstrap/state.ts                                      │
└─────────────────────────────────────────────────────────────┘
```

### Data Flow Architecture

```
User Input
  → QueryEngine.processUserInput()
  → query() [main loop in query.ts]
  → buildSystemPrompt() [queryContext.ts]
  → Claude API call [services/api/claude.ts]
  → Parse streaming response
  → If tool_use → runTools() [services/tools/toolOrchestration.ts]
      → executeTool() [services/tools/toolExecution.ts]
      → tool.call(input, context)
      → ToolResult
  → Append messages to session [sessionStorage.ts]
  → Stream events to UI
  → Repeat until stop_reason = "end_turn"
```

---

## 3. SUBSYSTEM CAPSULES

> **HOW TO USE**: For each task, load only the capsule README + its listed source files.
> Never load the entire `src/` at once.

### Implementation Order
```
01 → 02 → 04 → 05 → 06 → 03 → 07 → 08 → 09 → 10
```
(04 must come before 03 because agent spawning needs permission classification)

### Capsule Summary Table

| Capsule | Purpose | Key Source Files | Size | Load Strategy |
|---------|---------|-----------------|------|---------------|
| `01-runtime-session` | Session engine, message model, event queue | `query.ts`, `QueryEngine.ts`, `sessionStorage.ts`, `messages.ts` | LARGE | Load selectively |
| `02-tool-execution` | Tool registry, streaming execution, result storage | `Tool.ts`, `tools.ts`, `services/tools/*` | MEDIUM | Safe to load whole |
| `03-agent-orchestration` | Subagent spawn/resume, multi-agent swarm | `tools/AgentTool/*`, `tasks/*`, `utils/swarm/*` | VERY LARGE | Phase load |
| `04-bash-permissions` | Shell execution, risk classification | `tools/BashTool/*`, `utils/permissions/*` | VERY LARGE | Phase load |
| `05-memory-compaction` | Memory records, context compression | `memdir/*`, `services/compact/*`, `services/SessionMemory/*` | MEDIUM | Safe, 1-2 prompts |
| `06-mcp-integrations` | MCP server connections, tool discovery | `services/mcp/*`, `tools/MCPTool/*`, `plugins/*` | LARGE | Load selectively |
| `07-remote-bridge` | Remote sessions, WebSocket transport | `bridge/*`, `remote/*`, `server/*`, `upstreamproxy/*` | LARGE | Only if needed |
| `08-cli-commands-sdk` | Command registry, SDK schemas | `commands.ts`, `entrypoints/*`, `cli/*`, `schemas/*` | SMALL-MED | Safe |
| `09-security-auth-policy` | Auth, OAuth, policy limits, security | `utils/auth.ts`, `services/oauth/*`, `services/api/client.ts` | MEDIUM | Safe |
| `10-platform-utilities` | Logging, path, JSON, git, async | `utils/log.ts`, `utils/path.ts`, `utils/git/*` | MEDIUM | Load per-import |

### Required Core Interfaces (all chunks depend on)

```typescript
interface AppRuntimeSession {
  id: string;
  cwd: string;
  messages: RuntimeMessage[];
  tools: ToolRegistry;
  permissions: PermissionController;
  memory: MemoryController;
}
interface ToolRegistry { register(tool): void; get(name): ToolDefinition | undefined; list(): ToolDefinition[]; }
interface PermissionController { canRun(request: PermissionRequest): Promise<PermissionDecision>; }
interface AgentController { spawn(request): Promise<AgentHandle>; resume(id): Promise<AgentHandle>; stop(id): Promise<void>; }
interface MemoryController { search(query): Promise<MemoryRecord[]>; write(record): Promise<void>; compact(sessionId): Promise<CompactResult>; }
```

---

## 4. EXECUTION FLOW

### Startup (src/entrypoints/init.ts)
1. `init()` (memoized) — single-call initialization
2. `enableConfigs()` — load and validate configuration
3. `applySafeConfigEnvironmentVariables()` — env vars from settings
4. `setupGracefulShutdown()` — register cleanup handlers
5. `initialize1PEventLogging()` + GrowthBook analytics
6. `populateOAuthAccountInfoIfNeeded()` — prefetch auth
7. `initializePolicyLimitsLoadingPromise()` — remote policy fetch
8. `initializeRemoteManagedSettingsLoadingPromise()` — managed settings
9. OpenTelemetry telemetry (lazy-loaded ~1.1MB)

### Session Creation
```
src/utils/sessionStart.ts — creates new session
src/utils/sessionRestore.ts — loads existing session
src/bootstrap/state.ts — holds global session state (cwd, costs, model usage, etc.)
```

### Main Query Loop (src/query.ts, ~1730 lines)
```
query(input) — exported from query.ts
  ├── handleStopHooks() [query/stopHooks.ts]
  ├── buildQueryConfig() [query/config.ts]
  ├── createBudgetTracker() [query/tokenBudget.ts]
  ├── getAttachmentMessages() [utils/attachments.ts] — memory prefetch
  ├── isAutoCompactEnabled() → autoCompact flow [services/compact/autoCompact.ts]
  ├── API call via productionDeps.claude() [services/api/claude.ts]
  ├── Stream parsing → StreamEvent[]
  ├── If tool_use → StreamingToolExecutor [services/tools/StreamingToolExecutor.ts]
  ├── runTools() [services/tools/toolOrchestration.ts]
  ├── applyToolResultBudget() [utils/toolResultStorage.ts]
  ├── recordContentReplacement() [utils/sessionStorage.ts]
  └── Continue loop or yield Terminal
```

### QueryEngine (src/QueryEngine.ts, ~1296 lines)
The stateful wrapper around `query()`. Manages:
- `processUserInput()` — transforms raw input to messages
- `runQuery()` — executes the turn with full context
- Session persistence + transcript writing
- File state cache management
- Model override + thinking configuration
- Orphaned permission handling
- Session mode (coordinator/normal) matching

---

## 5. CORE FILES QUICK REFERENCE

| File | Role | Size | Key Exports |
|------|------|------|-------------|
| `src/bootstrap/state.ts` | Global singleton state — NEVER add state without reason | 56KB / 1759L | `getSessionId`, `getCwd`, `getTotalCost`, `getMainLoopModel`, `switchSession`, `setMeter` |
| `src/query.ts` | Main LLM loop, token budget, compact decisions | 68KB / 1730L | `query(input, deps, signal)` |
| `src/QueryEngine.ts` | Stateful session wrapper | 46KB / 1296L | `QueryEngine` class |
| `src/Tool.ts` | Tool interface + ToolUseContext | 29KB / 793L | `Tool`, `ToolUseContext`, `ValidationResult`, `QueryChainTracking` |
| `src/tools.ts` | Tool registry + feature-gated tool loading | 17KB / 390L | `getTools()`, tool arrays |
| `src/commands.ts` | Slash command registry | 25KB / 755L | all slash commands |
| `src/ultraplan.tsx` | Multi-agent planning feature (CCR) | 66KB / 471L | `buildUltraplanPrompt()`, `ultraplan` command |
| `src/setup.ts` | First-run setup wizard | 20KB | `setup()` |
| `src/utils/messages.ts` | Message creation/manipulation — HUGE | 193KB / 5513L | `createUserMessage`, `normalizeMessagesForAPI`, `createAssistantAPIErrorMessage`, etc. |
| `src/utils/sessionStorage.ts` | Session persistence (JSONL transcript) | 180KB / 5106L | `recordTranscript`, `flushSessionStorage`, `loadSession` |
| `src/utils/auth.ts` | API key + OAuth + AWS auth resolution | 65KB / 2003L | `getAnthropicApiKey`, `getOauthAccountInfo` |
| `src/utils/attachments.ts` | Image/PDF/file attachments + memory prefetch | 127KB | `getAttachmentMessages`, `startRelevantMemoryPrefetch` |
| `src/utils/worktree.ts` | Git worktree creation/management | 49KB / 1520L | `createWorktree`, `removeWorktree` |
| `src/utils/stats.ts` | Analytics stats collection | 33KB | `recordStat`, `flushStats` |
| `src/utils/toolResultStorage.ts` | Large tool result persistence | 38KB | `applyToolResultBudget`, `recordContentReplacement` |
| `src/services/api/claude.ts` | Anthropic API client (streaming) | 125KB / 3420L | `streamQuery`, `accumulateUsage` |
| `src/services/api/errors.ts` | API error types + rate limit handling | 41KB / 1208L | error classes, `categorizeRetryableAPIError` |
| `src/services/compact/compact.ts` | Context compaction engine | 60KB | `buildPostCompactMessages`, `runCompact` |
| `src/services/mcp/client.ts` | MCP connection client | 119KB | `MCPClient`, connection lifecycle |
| `src/services/mcp/config.ts` | MCP server configuration | 51KB | MCP config loading + validation |
| `src/services/tools/toolExecution.ts` | Tool executor with hooks | 60KB | `executeTool` |
| `src/bridge/bridgeMain.ts` | Remote bridge main session loop | 115KB | `bridgeMain` |
| `src/bridge/replBridge.ts` | REPL bridge transport | 100KB | `ReplBridge` |
| `src/cost-tracker.ts` | Token + cost tracking per model | 10KB | `trackCost`, `getModelUsage` |
| `src/context.ts` | Context limit helpers | 6KB | `ESCALATED_MAX_TOKENS`, context checks |
| `src/history.ts` | Conversation history management | 14KB | `buildHistory`, session history |

---

## 6. ALL TOOLS

> Each tool dir has: `ToolName.ts` (main), `prompt.ts` (system prompt), optional `constants.ts`

### Core File System Tools
| Tool | Dir | Description | Key Permission |
|------|-----|-------------|----------------|
| `FileReadTool` | `tools/FileReadTool/` | Read file content with range support | read |
| `FileWriteTool` | `tools/FileWriteTool/` | Write/create files | write (needs approval) |
| `FileEditTool` | `tools/FileEditTool/` | Edit file with diff/replace | write (needs approval) |
| `GlobTool` | `tools/GlobTool/` | Find files by pattern | read |
| `GrepTool` | `tools/GrepTool/` | Search file content with ripgrep | read |
| `NotebookEditTool` | `tools/NotebookEditTool/` | Edit Jupyter notebooks | write (needs approval) |

### Execution Tools
| Tool | Dir | Description | Key Permission |
|------|-----|-------------|----------------|
| `BashTool` | `tools/BashTool/` | Shell command execution + safety | classified by bashPermissions.ts |
| `REPLTool` | `tools/REPLTool/` | Interactive REPL (ant-only) | execute |
| `LSPTool` | `tools/LSPTool/` | Language server protocol queries | read |

### Web Tools
| Tool | Dir | Description | Key Permission |
|------|-----|-------------|----------------|
| `WebFetchTool` | `tools/WebFetchTool/` | HTTP fetch with preapproved list | network |
| `WebSearchTool` | `tools/WebSearchTool/` | Web search integration | network |

### Agent / Multi-agent Tools
| Tool | Dir | Description | Key Permission |
|------|-----|-------------|----------------|
| `AgentTool` | `tools/AgentTool/` | Spawn/manage sub-agents | inherit parent |
| `SendMessageTool` | `tools/SendMessageTool/` | Inter-agent messaging | internal |
| `TeamCreateTool` | `tools/TeamCreateTool/` | Create agent team | coordinator mode |
| `TeamDeleteTool` | `tools/TeamDeleteTool/` | Delete agent team | coordinator mode |
| `SkillTool` | `tools/SkillTool/` | Load and run skill scripts | varies |

### Task Management Tools
| Tool | Dir | Description | Key Permission |
|------|-----|-------------|----------------|
| `TaskCreateTool` | `tools/TaskCreateTool/` | Create background tasks | internal |
| `TaskGetTool` | `tools/TaskGetTool/` | Get task status | internal |
| `TaskListTool` | `tools/TaskListTool/` | List all tasks | internal |
| `TaskUpdateTool` | `tools/TaskUpdateTool/` | Update task state | internal |
| `TaskStopTool` | `tools/TaskStopTool/` | Stop a running task | internal |
| `TaskOutputTool` | `tools/TaskOutputTool/` | Get task output | internal |
| `TodoWriteTool` | `tools/TodoWriteTool/` | Write todo lists | internal |

### MCP / Plugin Tools
| Tool | Dir | Description | Key Permission |
|------|-----|-------------|----------------|
| `MCPTool` | `tools/MCPTool/` | Call MCP server tools | server-defined |
| `McpAuthTool` | `tools/McpAuthTool/` | Authenticate MCP server | auth |
| `ListMcpResourcesTool` | `tools/ListMcpResourcesTool/` | List MCP server resources | read |
| `ReadMcpResourceTool` | `tools/ReadMcpResourceTool/` | Read MCP resource | read |

### Plan Mode Tools
| Tool | Dir | Description | Key Permission |
|------|-----|-------------|----------------|
| `EnterPlanModeTool` | `tools/EnterPlanModeTool/` | Enter plan-only mode | N/A |
| `ExitPlanModeTool` | `tools/ExitPlanModeTool/` | Exit plan mode + approve | N/A |
| `EnterWorktreeTool` | `tools/EnterWorktreeTool/` | Switch to git worktree | filesystem |
| `ExitWorktreeTool` | `tools/ExitWorktreeTool/` | Return from worktree | filesystem |

### System / Control Tools
| Tool | Dir | Description | Key Permission |
|------|-----|-------------|----------------|
| `ConfigTool` | `tools/ConfigTool/` | Read/write configuration | internal |
| `BriefTool` | `tools/BriefTool/` | Upload session brief (Kairos) | network |
| `AskUserQuestionTool` | `tools/AskUserQuestionTool/` | Pause and ask user | N/A |
| `ToolSearchTool` | `tools/ToolSearchTool/` | Semantic tool search | internal |
| `SleepTool` | `tools/SleepTool/` | Delay execution (proactive/kairos) | N/A |
| `RemoteTriggerTool` | `tools/RemoteTriggerTool/` | Remote event trigger | network |
| `ScheduleCronTool` | `tools/ScheduleCronTool/` | Schedule recurring task (cron) | internal |
| `SyntheticOutputTool` | `tools/SyntheticOutputTool/` | Synthetic agent output marker | internal |

### Feature-Gated Tools (conditional loading in tools.ts)
| Tool | Feature Flag | Description |
|------|-------------|-------------|
| `REPLTool` | `USER_TYPE === 'ant'` | Interactive REPL |
| `SleepTool` | `PROACTIVE` or `KAIROS` | Sleep/delay |
| `CronCreateTool/CronDeleteTool/CronListTool` | `AGENT_TRIGGERS` | Cron scheduling |
| `RemoteTriggerTool` | `AGENT_TRIGGERS_REMOTE` | Remote triggers |
| `MonitorTool` | `MONITOR_TOOL` | Monitoring |
| `SendUserFileTool` | `KAIROS` | Send file to user |
| `PushNotificationTool` | `KAIROS` or `KAIROS_PUSH_NOTIFICATION` | Push notifications |
| `SubscribePRTool` | `KAIROS_GITHUB_WEBHOOKS` | Subscribe to PR events |
| `OverflowTestTool` | `OVERFLOW_TEST_TOOL` | Testing overflow |
| `CtxInspectTool` | `CONTEXT_COLLAPSE` | Context inspection |
| `TerminalCaptureTool` | `TERMINAL_PANEL` | Terminal capture |
| `WebBrowserTool` | `WEB_BROWSER_TOOL` | Full browser automation |
| `VerifyPlanExecutionTool` | `CLAUDE_CODE_VERIFY_PLAN=true` | Plan verification |

### AgentTool Built-in Agents (tools/AgentTool/built-in/)
| Agent | File | Role |
|-------|------|------|
| `claudeCodeGuideAgent` | `claudeCodeGuideAgent.ts` | Help with Claude Code usage |
| `exploreAgent` | `exploreAgent.ts` | Explore codebase |
| `generalPurposeAgent` | `generalPurposeAgent.ts` | Default sub-agent |
| `planAgent` | `planAgent.ts` | Planning specialist |
| `verificationAgent` | `verificationAgent.ts` | Verify task completion |

---

## 7. ALL COMMANDS

> Slash commands loaded in `src/commands.ts`. Each has an `index.ts` in `src/commands/<name>/`.

### Always-Loaded Commands
| Command | File | Description |
|---------|------|-------------|
| `/clear` | `commands/clear/` | Clear conversation / caches |
| `/compact` | `commands/compact/` | Compact context with summary |
| `/config` | `commands/config/` | View/edit configuration |
| `/context` | `commands/context/` | Show context usage |
| `/cost` | `commands/cost/` | Show session cost |
| `/diff` | `commands/diff/` | Show current git diff |
| `/doctor` | `commands/doctor/` | Diagnose environment issues |
| `/files` | `commands/files/` | List attached files |
| `/help` | `commands/help/` | Show available commands |
| `/ide` | `commands/ide/` | Open IDE integrations |
| `/init` | `commands/init.ts` | Initialize project (CLAUDE.md) |
| `/login` | `commands/login/` | Authenticate with Anthropic |
| `/logout` | `commands/logout/` | Remove auth credentials |
| `/mcp` | `commands/mcp/` | MCP server management |
| `/memory` | `commands/memory/` | View/edit memory |
| `/onboarding` | `commands/onboarding/` | Onboarding flow |
| `/pr-comments` | `commands/pr_comments/` | Review PR comments |
| `/release-notes` | `commands/release-notes/` | Show release notes |
| `/rename` | `commands/rename/` | Rename session |
| `/resume` | `commands/resume/` | Resume previous session |
| `/review` | `commands/review.js` | Code review |
| `/session` | `commands/session/` | Session management |
| `/share` | `commands/share/` | Share session |
| `/skills` | `commands/skills/` | Manage skills |
| `/status` | `commands/status/` | Show agent status |
| `/tasks` | `commands/tasks/` | Task management |
| `/teleport` | `commands/teleport/` | Teleport to remote environment |
| `/theme` | `commands/theme/` | Change UI theme |
| `/usage` | `commands/usage/` | Show token usage |
| `/vim` | `commands/vim/` | Vim mode |
| `/commit` | `commands/commit.js` | Git commit |
| `/commit-push-pr` | `commands/commit-push-pr.ts` | Commit + push + PR |
| `/security-review` | `commands/security-review.js` | Security review |
| `/agents` | `commands/agents/` | Agent management |
| `/branch` | `commands/branch/` | Git branch management |

### Feature-Gated Commands
| Command | Feature Flag |
|---------|-------------|
| `/proactive` | `PROACTIVE` or `KAIROS` |
| `/brief` | `KAIROS` or `KAIROS_BRIEF` |
| `/assistant` | `KAIROS` |
| `/bridge` | `BRIDGE_MODE` |
| `/remoteControlServer` | `DAEMON` and `BRIDGE_MODE` |
| `/voice` | `VOICE_MODE` |
| `/force-snip` | `HISTORY_SNIP` |
| `/workflows` | `WORKFLOW_SCRIPTS` |
| `/web` | `CCR_REMOTE_SETUP` |
| `/ultraplan` | (always loaded via ultraplan.tsx) |
| `agents-platform` | `USER_TYPE === 'ant'` |

---

## 8. SERVICES

### src/services/api/
| File | Role | Key Function |
|------|------|-------------|
| `claude.ts` (125KB) | Main Anthropic streaming client | `streamQuery()`, `accumulateUsage()` |
| `errors.ts` (41KB) | All API error types + messages | `categorizeRetryableAPIError()`, `getRateLimitErrorMessage()` |
| `withRetry.ts` (28KB) | Retry logic + backoff | `withRetry()`, `FallbackTriggeredError` |
| `client.ts` (16KB) | HTTP client wrapper | `createClient()` |
| `logging.ts` (24KB) | API request/response logging | `logAPIRequest()` |
| `sessionIngress.ts` (17KB) | Session ingress auth | ingress validation |
| `filesApi.ts` (21KB) | Files API (uploads) | `uploadFile()` |
| `errors.ts` | Error message generation for UI | rate limit messages |

### src/services/mcp/
| File | Role |
|------|------|
| `client.ts` (119KB) | Full MCP protocol client |
| `config.ts` (51KB) | MCP server config loading/validation |
| `auth.ts` (88KB) | MCP OAuth and auth flows |
| `useManageMCPConnections.ts` (44KB) | React hook for connection state |
| `xaa.ts` (18KB) | XAA (external auth agent) |
| `types.ts` | MCP types: `MCPServerConnection`, `ServerResource` |

### src/services/compact/
| File | Role |
|------|------|
| `compact.ts` (60KB) | Core compaction: summarize + rebuild messages |
| `autoCompact.ts` (12KB) | Automatic compaction trigger logic |
| `microCompact.ts` (19KB) | Micro-compaction for individual tool results |
| `sessionMemoryCompact.ts` (21KB) | Memory-aware compaction |
| `prompt.ts` (16KB) | Compaction prompts |
| `apiMicrocompact.ts` | API-side microcompact |

### src/services/tools/
| File | Role |
|------|------|
| `toolExecution.ts` (60KB) | Core tool executor with pre/post hooks |
| `toolHooks.ts` (22KB) | Hook dispatch (pre-tool, post-tool, stop) |
| `StreamingToolExecutor.ts` (17KB) | Streaming progress emission |
| `toolOrchestration.ts` (5KB) | Parallel/sequential tool orchestration |

### src/services/oauth/
| File | Role |
|------|------|
| `client.ts` (18KB) | OAuth token management, refresh |
| `index.ts` (6KB) | OAuth entry point |
| `auth-code-listener.ts` (6KB) | Auth code callback listener |
| `getOauthProfile.ts` | Fetch user profile from token |

### Other Services
| Dir/File | Role |
|----------|------|
| `services/SessionMemory/` | Session-scoped memory storage |
| `services/AgentSummary/` | Agent completion summaries |
| `services/MagicDocs/` | Documentation generation |
| `services/PromptSuggestion/` | AI prompt suggestions |
| `services/autoDream/` | Proactive dream/suggestion system |
| `services/extractMemories/` | Extract memories from messages |
| `services/teamMemorySync/` | Shared team memory sync |
| `services/lsp/` | Language server management |
| `services/compact/` | Context compaction (see above) |
| `services/policyLimits/` | Enterprise policy limits |
| `services/remoteManagedSettings/` | Remote-managed settings fetch |
| `services/settingsSync/` | Settings synchronization |
| `services/toolUseSummary/` | Tool usage summary generation |
| `services/claudeAiLimits.ts` (16KB) | Claude.ai subscription limits |
| `services/diagnosticTracking.ts` | Diagnostic event tracking |
| `services/tokenEstimation.ts` (16KB) | Token count estimation |
| `services/rateLimitMessages.ts` (10KB) | Rate limit user messages |
| `services/notifier.ts` | Desktop notifications |
| `services/preventSleep.ts` | Prevent system sleep during tasks |

---

## 9. UTILS

> 138 files — load only what your task needs. Grouped by domain.

### Session & Query Core
| File | Role | Used By |
|------|------|---------|
| `sessionStorage.ts` (180KB) | JSONL transcript read/write | QueryEngine, all tools |
| `sessionStoragePortable.ts` (25KB) | Portable session storage | SDK, bridge |
| `sessionState.ts` | Session flags + per-session state | query.ts |
| `sessionRestore.ts` (20KB) | Load existing session + history | QueryEngine |
| `sessionStart.ts` (8KB) | Create new session | QueryEngine, entrypoints |
| `sessionActivity.ts` | Track session activity/idle | QueryEngine |
| `sessionEnvironment.ts` | Env vars for session | session init |
| `sessionEnvVars.ts` | Session-scoped env vars | session init |
| `sessionTitle.ts` | Auto-generate session title | sessionStorage |
| `sessionUrl.ts` | Session URL generation | sharing |
| `messages.ts` (193KB) | ALL message manipulation functions | query.ts, QueryEngine |
| `messageQueueManager.ts` (16KB) | Slash command queue + priorities | query.ts |
| `messagePredicates.ts` | Type predicates for messages | messages.ts |
| `queryContext.ts` (5KB) | Build system prompt parts | QueryEngine |
| `queryHelpers.ts` (19KB) | Normalize messages, handle permissions | query.ts |
| `QueryGuard.ts` | Prevent concurrent queries | QueryEngine |

### Auth & Configuration
| File | Role |
|------|------|
| `auth.ts` (65KB) | API key resolution: env → keychain → config → OAuth |
| `config.ts` | Global + project config R/W |
| `settings/settings.ts` | Settings resolution with source tracking |
| `managedEnv.ts` | Managed env var injection |
| `managedEnvConstants.ts` | Constant env var names |
| `envUtils.ts` | Env var helpers, platform detection |
| `lockfile.ts` | Config file locking |
| `secureStorage/` | Keychain + encrypted storage |

### Model & API
| File | Role |
|------|------|
| `api.ts` (26KB) | API formatting helpers, system prompt building |
| `tokens.ts` (9KB) | Token counting + estimation |
| `tokenBudget.ts` | Turn token budget management |
| `context.ts` | Context window limits, escalated limits |
| `modelCost.ts` | Per-model cost table |
| `thinking.ts` | Extended thinking config |
| `effort.ts` | Effort level resolution |
| `betas.ts` | Beta feature header management |

### Tool Utilities
| File | Role |
|------|------|
| `toolResultStorage.ts` (38KB) | Large result persistence + retrieval |
| `toolPool.ts` | Concurrent tool execution pool |
| `toolErrors.ts` | Tool error types + formatting |
| `toolSchemaCache.ts` | Cache tool JSON schemas |
| `toolSearch.ts` (26KB) | Semantic tool search |
| `attachments.ts` (127KB) | File/image/PDF attachment processing |
| `readFileInRange.ts` (12KB) | Efficient file range reading |
| `ripgrep.ts` (21KB) | Ripgrep execution wrapper |
| `Shell.ts` (16KB) | Shell abstraction |
| `ShellCommand.ts` (14KB) | Shell command building/parsing |
| `bash/` | Bash command parsing sub-utilities |

### Memory
| File | Role |
|------|------|
| `memory/` | Memory storage helpers |
| `memoryFileDetection.ts` (10KB) | Detect CLAUDE.md memory files |
| `agenticSessionSearch.ts` (10KB) | Search previous sessions |
| `analyzeContext.ts` (42KB) | Analyze conversation context |

### Multi-Agent / Swarm
| File | Role |
|------|------|
| `swarm/` | Multi-agent swarm coordination |
| `swarm/inProcessRunner.ts` (53KB) | In-process agent runner |
| `swarm/permissionSync.ts` (26KB) | Sync permissions across agents |
| `swarm/teamHelpers.ts` (21KB) | Team creation/management |
| `agentContext.ts` (6KB) | Per-agent context isolation |
| `agentId.ts` | Agent ID generation/parsing |
| `teammate.ts` (9KB) | Teammate lifecycle |
| `teammateMailbox.ts` (33KB) | Inter-agent message passing |
| `teamDiscovery.ts` | Find/list teams |
| `teamMemoryOps.ts` | Team memory R/W |

### Git & GitHub
| File | Role |
|------|------|
| `git.ts` | Git operations |
| `git/` | Git sub-utilities (config parser, filesystem) |
| `github/` | GitHub API integration |
| `worktree.ts` (49KB) | Git worktree management |
| `plans.ts` (12KB) | Plan file management |

### Permissions
| File | Role |
|------|------|
| `permissions/bashClassifier.ts` | Classify bash command safety |
| `permissions/dangerousPatterns.ts` | Dangerous command patterns |
| `permissions/yoloClassifier.ts` (52KB) | Full permission classifier |
| `hooks/toolPermission/` | Tool permission decision hooks |

### Platform & System
| File | Role |
|------|------|
| `platform.ts` | OS/platform detection |
| `path.ts` | Path helpers + traversal protection |
| `process.ts` | Process helpers |
| `systemDirectories.ts` | System directory resolution |
| `xdg.ts` | XDG base directory spec |
| `windowsPaths.ts` | Windows-specific paths |
| `tmuxSocket.ts` (13KB) | Tmux socket management |
| `computerUse/` | Computer use (GUI automation) |

### Async & Control Flow
| File | Role |
|------|------|
| `abortController.ts` | Cancellation token management |
| `signal.ts` | Reactive signal utility |
| `sleep.ts` | Sleep + cancellable sleep |
| `timeouts.ts` | Timeout helpers |
| `sequential.ts` | Sequential async helper |
| `queueProcessor.ts` | Queue with concurrency control |
| `memoize.ts` (8KB) | Memoize with TTL + async |

### Serialization & Data
| File | Role |
|------|------|
| `json.ts` (9KB) | JSON parse/stringify helpers |
| `jsonRead.ts` | JSON file reading |
| `markdown.ts` (11KB) | Markdown processing |
| `xml.ts` | XML tag helpers |
| `yaml.ts` | YAML parsing |
| `stringUtils.ts` | String manipulation |
| `truncate.ts` | Text truncation |
| `treeify.ts` | Convert paths to tree display |

### Logging & Diagnostics
| File | Role |
|------|------|
| `log.ts` (11KB) | Main logger (logError, logEvent, captureAPIRequest) |
| `debug.ts` | Debug logging (logForDebugging, logAntError) |
| `diagLogs.ts` | Diagnostic-only logging (no PII) |
| `stats.ts` (33KB) | Metrics collection |
| `statsCache.ts` (13KB) | Cached metrics |

### Special Features
| File | Role |
|------|------|
| `ultraplan/ccrSession.ts` | Poll CCR for plan approval |
| `ultraplan/keyword.ts` | Ultraplan keyword detection |
| `teleport/` | Remote environment teleportation |
| `todo/types.ts` | Todo list types |
| `task/framework.ts` | Task state management framework |
| `filePersistence/` | File persistence utilities |

---

## 10. TYPES & SCHEMAS

### Core Type Files
| File | Key Types |
|------|-----------|
| `src/types/command.ts` (7KB) | `Command`, `CommandContext`, `LocalJSXCommandCall` |
| `src/types/hooks.ts` (9KB) | `HookEvent`, `PromptRequest`, `PromptResponse`, `HookProgress` |
| `src/types/ids.ts` | `SessionId`, `AgentId`, tagged ID types |
| `src/types/logs.ts` (11KB) | `Entry`, `SerializedMessage`, `TranscriptMessage`, log format |
| `src/types/permissions.ts` (13KB) | `PermissionMode`, `PermissionResult`, `AdditionalWorkingDirectory` |
| `src/types/plugin.ts` (11KB) | `Plugin`, `PluginHookMatcher`, plugin lifecycle |
| `src/entrypoints/agentSdkTypes.ts` (13KB) | `SDKMessage`, `SDKStatus`, `PermissionMode`, SDK event types |
| `src/entrypoints/sandboxTypes.ts` (5KB) | Sandbox environment types |

### Message Types (spread across codebase, defined in messages.ts)
```typescript
// Core message union (from messages.ts imports)
type Message =
  | UserMessage
  | AssistantMessage
  | SystemMessage
  | SystemCompactBoundaryMessage
  | SystemAPIErrorMessage
  | SystemInformationalMessage
  | SystemBridgeStatusMessage
  | SystemMemorySavedMessage
  | SystemPermissionRetryMessage
  | SystemStopHookSummaryMessage
  | SystemAwaySummaryMessage
  | SystemTurnDurationMessage
  | SystemScheduledTaskFireMessage
  | SystemAgentsKilledMessage
  | SystemApiMetricsMessage
  | AttachmentMessage
  | ProgressMessage
  | TombstoneMessage
  | ToolUseSummaryMessage
  | RequestStartEvent
  | StreamEvent
```

### Global State (bootstrap/state.ts)
```typescript
// Key state fields accessed across codebase
state = {
  originalCwd, projectRoot, cwd,
  totalCostUSD, totalAPIDuration, totalToolDuration,
  turnHookDurationMs, turnToolDurationMs, turnToolCount,
  totalLinesAdded, totalLinesRemoved,
  modelUsage: { [modelName]: ModelUsage },
  mainLoopModelOverride, initialMainLoopModel, modelStrings,
  isInteractive, kairosActive,
  sessionId, sessionProjectDir,
  isNonInteractiveSession,
  // + OpenTelemetry meter, tracerProvider, loggerProvider
}
```

---

## 11. FEATURE FLAGS

> All gated via `feature('FLAG_NAME')` from `bun:bundle`. Dead-code-eliminated at build time.

| Flag | What It Unlocks |
|------|----------------|
| `REACTIVE_COMPACT` | Reactive (event-driven) context compaction |
| `CONTEXT_COLLAPSE` | Aggressive context collapse + `CtxInspectTool` |
| `COORDINATOR_MODE` | Multi-agent coordinator mode + `CLAUDE_CODE_COORDINATOR_MODE` env |
| `HISTORY_SNIP` | Snip old history entries |
| `BG_SESSIONS` | Background session task summaries |
| `EXPERIMENTAL_SKILL_SEARCH` | Semantic skill search + prefetch |
| `TEMPLATES` | Job template classifier |
| `PROACTIVE` | Proactive sleep/monitoring tools |
| `KAIROS` | Full Kairos feature set (mobile, push, brief) |
| `KAIROS_BRIEF` | Brief upload tool only |
| `KAIROS_PUSH_NOTIFICATION` | Push notifications only |
| `KAIROS_GITHUB_WEBHOOKS` | GitHub PR subscription |
| `BRIDGE_MODE` | Remote bridge mode |
| `DAEMON` | Daemon process mode |
| `VOICE_MODE` | Voice input |
| `WORKFLOW_SCRIPTS` | Workflow automation scripts |
| `AGENT_TRIGGERS` | Cron-based agent triggers |
| `AGENT_TRIGGERS_REMOTE` | Remote event triggers |
| `MONITOR_TOOL` | System monitoring tool |
| `OVERFLOW_TEST_TOOL` | Testing tool overflow handling |
| `TERMINAL_PANEL` | Terminal panel capture |
| `WEB_BROWSER_TOOL` | Full browser automation |
| `CCR_REMOTE_SETUP` | CCR remote setup command |

### Environment Variable Feature Gates
| Env Var | Effect |
|---------|--------|
| `USER_TYPE=ant` | Enables internal Anthropic tools (REPLTool, etc.) |
| `CLAUDE_CODE_COORDINATOR_MODE=1` | Enables coordinator mode |
| `CLAUDE_CODE_VERIFY_PLAN=true` | Enables plan verification tool |
| `ANTHROPIC_API_KEY` | Direct API key |
| `ANTHROPIC_BASE_URL` | Override API base URL |
| `CLAUDE_CODE_MAX_OUTPUT_TOKENS` | Override max output tokens |

---

## 12. KEY DEPENDENCY GRAPH

```
bootstrap/state.ts
  ← everything (global state, never circular)

query.ts
  ← QueryEngine.ts
  ← services/api/claude.ts
  ← services/tools/toolOrchestration.ts
  ← services/compact/autoCompact.ts
  ← utils/messages.ts
  ← utils/attachments.ts
  ← utils/messageQueueManager.ts
  ← utils/sessionStorage.ts
  ← utils/queryHelpers.ts
  ← memdir/memdir.ts
  ← query/stopHooks.ts, query/config.ts, query/tokenBudget.ts

QueryEngine.ts
  ← query.ts
  ← utils/processUserInput/processUserInput.ts
  ← utils/queryContext.ts
  ← utils/fileHistory.ts
  ← utils/fileStateCache.ts
  ← coordinator/coordinatorMode.ts (feature-gated)

Tool.ts (interface only — no impl deps)
  ← tools.ts
  ← all tools/*

tools.ts → registers all tools (circular-safe via lazy require)
  ← tools/AgentTool/AgentTool.tsx
  ← tools/BashTool/BashTool.tsx
  ← ... all other tools

services/tools/toolExecution.ts
  ← Tool.ts
  ← services/tools/toolHooks.ts
  ← utils/toolResultStorage.ts
  ← utils/toolErrors.ts

services/api/claude.ts
  ← utils/api.ts
  ← utils/messages.ts
  ← utils/betas.ts
  ← utils/model/providers.ts
  ← utils/auth.ts

utils/auth.ts
  ← services/oauth/client.ts
  ← utils/secureStorage/
  ← utils/config.ts

services/mcp/client.ts
  ← services/mcp/types.ts
  ← services/mcp/auth.ts
  ← utils/mcp/

entrypoints/init.ts (no circular deps)
  ← services/lsp/manager.ts
  ← services/oauth/client.ts
  ← services/policyLimits/
  ← services/remoteManagedSettings/
  ← utils/config.ts
```

### Circular Dependency Management
The codebase uses these patterns to break cycles:
1. **Lazy `require()`**: `const getTeamCreateTool = () => require('./tools/TeamCreateTool/...')`
2. **Type-only imports**: `import type { X }` avoids value-level cycles
3. **Centralized type files**: `types/permissions.ts`, `types/tools.ts` are leaves with no upstream deps
4. **Feature flags**: `feature('X') ? require('./optional.js') : null`

---

## 13. WEB PLATFORM MIGRATION GUIDE

### What to KEEP (port directly)
| Subsystem | Reasoning |
|-----------|-----------|
| Tool definitions (`src/tools/*/prompt.ts`) | LLM prompts are UI-agnostic |
| Tool validation logic | Business rules, reusable |
| Session message model | Core data model |
| Memory + compact logic | Service layer, not UI |
| Permission classification | Security-critical, keep |
| MCP client | Protocol layer |
| Auth logic (token, OAuth flows) | Core auth patterns |
| Cost tracker | Model-agnostic |

### What to REPLACE (CLI → Web)
| CLI Component | Web Replacement |
|---------------|----------------|
| `src/cli/structuredIO.ts` | WebSocket / SSE event stream |
| `src/cli/transports/` | HTTP transport layer |
| `src/bootstrap/state.ts` (process singletons) | Server-side session store (Redis/DB) |
| `src/utils/Shell.ts` + BashTool execution | Sandboxed container or server-side exec |
| `bun:bundle` feature flags | Environment variables + config API |
| Filesystem-based session storage (JSONL) | Database (PostgreSQL, SQLite, etc.) |
| `src/utils/auth.ts` keychain | Browser session / JWT tokens |
| CLI notification (`services/notifier.ts`) | Browser Notification API |
| `preventSleep.ts` | No equivalent needed |
| `tmuxSocket.ts` | Terminal websocket (xterm.js) |

### Multi-Provider Model Support (beyond Claude)
The codebase already has provider abstraction at:
- `src/utils/model/providers.ts` — `getAPIProvider()`, `isFirstPartyAnthropicBaseUrl()`
- `src/utils/model/model.ts` — `getMainLoopModel()`, `parseUserSpecifiedModel()`
- `src/utils/model/modelStrings.ts` — `getModelStrings()` per provider
- `src/utils/model/configs.ts` — `ALL_MODEL_CONFIGS` (opus, sonnet, etc.)
- `src/services/api/claude.ts` — handles Anthropic, Bedrock, Vertex

**To add OpenAI / Gemini / Mistral / Groq**:
1. Add provider enum to `providers.ts`
2. Create `services/api/openai.ts` (parallel to `claude.ts`)
3. Route in `query.ts` based on `getAPIProvider()`
4. Map tool schemas (OpenAI uses different format than Anthropic)
5. Normalize streaming events to common `StreamEvent` type

### Recommended Web Architecture
```
┌─────────────────────────────────────────────────────┐
│  Browser UI (React + WebSockets)                     │
│  - Chat panel, tool status, file tree, terminal     │
├─────────────────────────────────────────────────────┤
│  API Server (Express / Fastify / Hono)              │
│  - /api/session  /api/messages  /api/tools          │
│  - SSE / WebSocket for streaming                    │
├─────────────────────────────────────────────────────┤
│  Ported Business Logic (from src/)                  │
│  - QueryEngine (adapted)                            │
│  - Tool Registry + Executors                        │
│  - Memory + Compact                                 │
│  - Permission Controller                            │
│  - MCP Client                                       │
│  - Auth Controller                                  │
├─────────────────────────────────────────────────────┤
│  Infrastructure                                      │
│  - Session DB (PostgreSQL / SQLite)                 │
│  - File Storage (local FS / S3)                     │
│  - Execution Sandbox (Docker / Firecracker)         │
│  - Redis (session state, pub/sub for streaming)     │
└─────────────────────────────────────────────────────┘
```

### Subsystem Implementation Order for Web Platform
Follow the same order as `ai-subsystems/INSTRUCTIONS.md`:
```
01 → 02 → 04 → 05 → 06 → 03 → 07 → 08 → 09 → 10
```
Then add web-specific layers:
```
11. HTTP API routes (wrap 01 session contract)
12. WebSocket/SSE streaming adapter (wrap query() events)
13. Browser auth (OAuth PKCE, JWT session)
14. Multi-provider router (OpenAI, Gemini, etc.)
15. Sandbox execution backend (Docker, etc.)
16. Web UI (React, terminal emulator, file browser)
```

---

## HOW TO USE THIS NAVIGATOR

### For implementing a specific feature:
1. Find the feature in sections 6-9
2. Note the file paths
3. Load `ai-subsystems/<N>/README.md` for that chunk
4. Load only the source files listed

### For debugging a bug:
1. Trace through the execution flow in section 4
2. Identify which layer the bug is in
3. Load only that layer's files

### For adding a new tool:
1. Create `src/tools/MyTool/MyTool.ts` (implements `Tool` interface from `src/Tool.ts`)
2. Create `src/tools/MyTool/prompt.ts` (system prompt + schema)
3. Register in `src/tools.ts` `getTools()` function
4. Load `02-tool-execution/README.md` as your reference

### For adding a new model provider:
1. Load `09-security-auth-policy/README.md` for auth patterns
2. Load `src/services/api/claude.ts` (first 200 lines as template)
3. Load `src/utils/model/providers.ts` (small, safe to load whole)

### For adding a new command:
1. Create `src/commands/mycommand/index.ts`
2. Import in `src/commands.ts`
3. Load `08-cli-commands-sdk/README.md` for patterns

---

*Navigator generated from codebase analysis — 592 TS files, 18 TSX files, 138 utils files.*
*For web migration, always read `ai-subsystems/INSTRUCTIONS.md` first.*
