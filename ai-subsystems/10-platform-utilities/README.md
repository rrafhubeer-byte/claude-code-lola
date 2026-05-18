# 10 Platform Utilities

Purpose: provide shared utilities used by all other chunks.

Approximate source size: medium. Load only the utilities directly imported by the chunk being implemented.

## Source Paths

- `src/utils/path.ts`
- `src/utils/process.ts`
- `src/utils/platform.ts`
- `src/utils/log.ts`
- `src/utils/json.ts`
- `src/utils/jsonRead.ts`
- `src/utils/sleep.ts`
- `src/utils/timeouts.ts`
- `src/utils/sequential.ts`
- `src/utils/queueProcessor.ts`
- `src/utils/abortController.ts`
- `src/utils/signal.ts`
- `src/utils/tempfile.ts`
- `src/utils/uuid.ts`
- `src/utils/taggedId.ts`
- `src/utils/stringUtils.ts`
- `src/utils/markdown.ts`
- `src/utils/xml.ts`
- `src/utils/yaml.ts`
- `src/utils/zodToJsonSchema.ts`
- `src/utils/git/*`
- `src/utils/github/*`
- `src/utils/filePersistence/*`
- `src/utils/task/*`
- `src/utils/native-ts/*`
- `src/services/internalLogging.ts`
- `src/services/diagnosticTracking.ts`
- `src/services/notifier.ts`
- `src/services/preventSleep.ts`

## Implement

- Logging facade.
- File/path helpers.
- JSON/YAML/XML helpers.
- Async queue/sequential helpers.
- Abort/signal helpers.
- Git filesystem helpers.
- Task output storage helpers.
- Diagnostic/notifier placeholders.

## Required Placeholders

- `TODO_SUBSYSTEM(10-platform-utilities): target app logger`
- `TODO_SUBSYSTEM(10-platform-utilities): notification provider`
- `TODO_SUBSYSTEM(10-platform-utilities): native binary bindings`
- `TODO_SUBSYSTEM(10-platform-utilities): git provider fallback`
- `TODO_SUBSYSTEM(10-platform-utilities): platform-specific path rules`

## Public Contract

Expose utilities through one import surface such as:

- `platform`
- `logger`
- `fileSystem`
- `asyncQueue`
- `gitUtils`
- `serialization`

Avoid letting feature chunks import random utility internals directly.

