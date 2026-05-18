# 04 Bash Permissions

Purpose: implement shell command execution and risk classification.

Approximate source size: very large. Load in phases.

## Source Paths

Shell tool:

- `src/tools/BashTool/BashTool.tsx`
- `src/tools/BashTool/bashCommandHelpers.ts`
- `src/tools/BashTool/bashPermissions.ts`
- `src/tools/BashTool/bashSecurity.ts`
- `src/tools/BashTool/commandSemantics.ts`
- `src/tools/BashTool/destructiveCommandWarning.ts`
- `src/tools/BashTool/modeValidation.ts`
- `src/tools/BashTool/pathValidation.ts`
- `src/tools/BashTool/readOnlyValidation.ts`
- `src/tools/BashTool/sedEditParser.ts`
- `src/tools/BashTool/sedValidation.ts`
- `src/tools/BashTool/shouldUseSandbox.ts`
- `src/tools/BashTool/toolName.ts`
- `src/tools/BashTool/utils.ts`

Parsing and classification:

- `src/utils/bash/*`
- `src/utils/permissions/bashClassifier.ts`
- `src/utils/permissions/dangerousPatterns.ts`
- `src/utils/permissions/yoloClassifier.ts`
- `src/hooks/toolPermission/*`

## Implement

- Shell command tool.
- Command parser interface.
- Read-only command classifier.
- Dangerous command classifier.
- Permission request flow.
- Sed/edit command detection.
- Optional sandbox decision boundary.

## Required Placeholders

- `TODO_SUBSYSTEM(04-bash-permissions): actual shell process runner`
- `TODO_SUBSYSTEM(04-bash-permissions): sandbox backend`
- `TODO_SUBSYSTEM(04-bash-permissions): user permission prompt UI`
- `TODO_SUBSYSTEM(04-bash-permissions): OS-specific shell provider`
- `TODO_SUBSYSTEM(04-bash-permissions): command audit logger`

## Public Contract

Expose:

- `classifyShellCommand(command, context)`
- `requestShellPermission(command, classification)`
- `runShellCommand(command, context)`
- `ShellExecutionResult`

Never wire shell execution directly to model output. It must pass through classification and permission control.

