# About this site

This field guide maps the current `letta-code` harness and the `letta-agent-sdk` client to the code that shapes them. It gives readers a compact mental model of the v2 Letta system: what each part does, how the parts fit together, and where the behavior lives in the repositories.

The official documentation at https://docs.letta.com covers quickstarts, feature how-tos, configuration, and protocol or API reference. This guide complements that material by staying at the system level. It keeps a reader close to the implementation without carrying old assumptions from earlier versions.

## Who this is for

This guide is for engineers adopting Letta, embedding it into another product, extending the harness, or contributing to the SDK or app-server boundary. It also serves readers who remember the MemGPT-era v1 server and need the current v2 model without treating that older design as current architecture.

## How it was made

Doc Holiday generated this guide by directly exploring the current `letta-code` and `letta-agent-sdk` repositories. The write-up follows the live code paths that explain the platform framing, including `src/agent/memory-filesystem.ts` (`resolveScopedMemoryDir`, `applyMemfsFlags`), `src/queue/turn-queue-runtime.ts` (`mergeQueuedTurnInput`), `src/channels/registry.ts` (`ChannelRegistry`, `initializeChannels`, `completePairing`), `src/mods/mod-engine.ts` (`createModEngine`, `loadLocalMods`), `src/app-server-client.ts` (`AppServerClient`, `runTurn`), `src/websocket/listen-client.ts` (`startListenerClient`), and the SDK files `letta-agent-sdk/src/protocol.ts`, `letta-agent-sdk/src/session.ts`, and `letta-agent-sdk/src/app-server-session.ts`.

## Scope and honesty

This page and the rest of the field guide capture a dated snapshot of the v2 codebase. Some paths still change or remain partial, so the guide treats them as observed facts at the time of generation rather than permanent architecture. The official docs remain authoritative for user-facing guidance, and corrections are welcome when the code changes or this snapshot drifts.

Generation stamp: latest observed `letta-code` main commit `5c236cb` dated `2026-07-20`.
Contact / repo note: [contact/repo note]

## Table of contents

- [00 The Big Picture](./00-the-big-picture.md)
- [01 Memory and MemFS](./01-memory-and-memfs.md)
- [02 Turn Queueing](./02-turn-queueing.md)
- [03 Channels and Pairing](./03-channels-and-pairing.md)
- [04 Mods](./04-mods.md)
- [05 App Server Client](./05-app-server-client.md)
- [06 SDK Sessions](./06-sdk-sessions.md)
- [07 Protocol Surface](./07-protocol-surface.md)
- [08 Drift and Limits](./08-drift-and-limits.md)

## Where to look in the code

- `README.md` in `letta-code` explains the public harness framing and current product model.
- `src/agent/memory-filesystem.ts`, `src/queue/turn-queue-runtime.ts`, and `src/channels/registry.ts` in `letta-code` show memory directory handling, turn merging, routing, and pairing.
- `src/mods/mod-engine.ts` in `letta-code` describes local mod loading, diagnostics, and lifecycle.
- `src/app-server-client.ts` and `src/websocket/listen-client.ts` in `letta-code` show app-server turn flow and the listener entrypoint.
- `README.md`, `src/protocol.ts`, `src/session.ts`, and `src/app-server-session.ts` in `letta-agent-sdk` define the SDK surface, session flow, and the app-server boundary.