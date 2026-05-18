# 08 CLI Commands And SDK

Purpose: expose core mechanics through commands and programmatic SDK entrypoints.

Approximate source size: small to medium.

## Source Paths

Commands:

- `src/commands.ts`
- `src/commands/agents/*`
- `src/commands/branch/*`
- `src/commands/clear/*`
- `src/commands/compact/*`
- `src/commands/config/*`
- `src/commands/context/*`
- `src/commands/files/*`

SDK:

- `src/entrypoints/*`
- `src/entrypoints/sdk/*`
- `src/cli/handlers/*`
- `src/cli/exit.ts`
- `src/cli/ndjsonSafeStringify.ts`
- `src/types/*`
- `src/schemas/*`

## Implement

- Command registry.
- Command argument parsing boundary.
- Programmatic SDK schemas.
- Non-interactive structured IO.
- Clear/config/context/files commands.
- SDK event schema compatibility.

## Required Placeholders

- `TODO_SUBSYSTEM(08-cli-commands-sdk): target app command router`
- `TODO_SUBSYSTEM(08-cli-commands-sdk): CLI renderer`
- `TODO_SUBSYSTEM(08-cli-commands-sdk): SDK transport adapter`
- `TODO_SUBSYSTEM(08-cli-commands-sdk): generated schema refresh pipeline`

## Public Contract

Expose:

- `CommandRegistry`
- `registerCoreCommands(registry)`
- `runCommand(name, args, context)`
- `SdkControlSchemas`
- `SdkCoreSchemas`

