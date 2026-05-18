# 07 Remote Bridge

Purpose: support remote sessions, bridge transport, direct connect, and background session control.

Approximate source size: large. Load only if your app needs remote or mobile/background control.

## Source Paths

- `src/bridge/*`
- `src/remote/*`
- `src/server/*`
- `src/upstreamproxy/*`
- `src/cli/transports/*`
- `src/cli/remoteIO.ts`
- `src/cli/structuredIO.ts`
- `src/utils/background/remote/*`
- `src/utils/teleport/*`

## Implement

- Bridge config.
- Session creation over bridge.
- Message transport abstraction.
- Remote permission bridge.
- WebSocket/SSE transport boundary.
- Remote session manager.
- Direct connect session manager.

## Required Placeholders

- `TODO_SUBSYSTEM(07-remote-bridge): remote service URL and auth`
- `TODO_SUBSYSTEM(07-remote-bridge): WebSocket server/client adapter`
- `TODO_SUBSYSTEM(07-remote-bridge): reconnect and backoff policy`
- `TODO_SUBSYSTEM(07-remote-bridge): remote permission prompt relay`
- `TODO_SUBSYSTEM(07-remote-bridge): mobile/background wake mechanism`

## Public Contract

Expose:

- `RemoteSessionManager`
- `BridgeTransport`
- `createRemoteSession(request)`
- `sendRemoteMessage(sessionId, message)`
- `subscribeRemoteEvents(sessionId, handler)`

