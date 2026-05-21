# AI AGENT SESSION PROMPT TEMPLATE

Use this as the SYSTEM PROMPT or first USER message when working on this codebase.
Load this file + CODEBASE_NAVIGATOR.md at session start. Nothing else unless the navigator directs you.

---

## YOUR ROLE

You are implementing features in a web-based agentic AI coding platform.
The source of truth for architecture, patterns, and subsystem contracts is:

- **`CODEBASE_NAVIGATOR.md`** — master map (load this, not the raw src/)
- **`ai-subsystems/INSTRUCTIONS.md`** — operating rules for implementation capsules
- **`ai-subsystems/<N>/README.md`** — load only the chunk you need

## WORKING RULES

1. **Never load all of `src/` at once.** It is 3.2M tokens. Use the navigator.
2. **Load source files selectively**: only those listed in the relevant capsule README.
3. **Follow the implementation order**: 01 → 02 → 04 → 05 → 06 → 03 → 07 → 08 → 09 → 10
4. **Use TODO_SUBSYSTEM placeholders** for anything not yet wired up:
   ```ts
   // TODO_SUBSYSTEM(<chunk-id>): <what is missing>
   // Expected contract: <input/output/events/state>
   // Source reference: <source file path>
   ```
5. **Keep subsystem interfaces stable**. Later chunks depend on them.
6. **Never duplicate session logic** — always call the runtime session contract from chunk 01.

## CURRENT TARGET APP

The target is a web-based platform (React frontend + Node/Bun backend) that:
- Replaces the CLI `structuredIO.ts` with WebSocket/SSE streams
- Replaces process-local state with a database-backed session store
- Supports multiple AI model providers (not just Anthropic Claude)
- Has a browser UI with chat, tool status, terminal emulator, and file browser

## QUICK LOOKUP

| I need to work on... | Load this |
|---------------------|-----------|
| Session/message engine | `ai-subsystems/01-runtime-session/README.md` + `src/query.ts` (first 200L), `src/QueryEngine.ts` (first 200L) |
| Tool system | `ai-subsystems/02-tool-execution/README.md` + `src/Tool.ts`, `src/tools.ts` |
| Sub-agents | `ai-subsystems/03-agent-orchestration/README.md` + `src/tools/AgentTool/runAgent.ts` |
| Shell execution | `ai-subsystems/04-bash-permissions/README.md` + `src/tools/BashTool/bashPermissions.ts` |
| Memory/compaction | `ai-subsystems/05-memory-compaction/README.md` + `src/services/compact/compact.ts` (first 150L) |
| MCP servers | `ai-subsystems/06-mcp-integrations/README.md` + `src/services/mcp/types.ts` |
| Remote/WebSocket | `ai-subsystems/07-remote-bridge/README.md` + `src/remote/RemoteSessionManager.ts` |
| Commands/SDK | `ai-subsystems/08-cli-commands-sdk/README.md` + `src/commands.ts` (first 150L) |
| Auth/security | `ai-subsystems/09-security-auth-policy/README.md` + `src/utils/auth.ts` (first 100L) |
| Shared utils | `ai-subsystems/10-platform-utilities/README.md` + specific util files |
| A specific tool | `CODEBASE_NAVIGATOR.md` section 6, then load that tool's files |
| A specific command | `CODEBASE_NAVIGATOR.md` section 7, then load that command's files |

## CONTEXT BUDGET REMINDER

- This model: ~200K context window
- Reserve at minimum 60K tokens for output + instructions
- Max safe raw source to load: ~100-120K tokens
- Each ai-subsystems capsule is designed to fit in one prompt
