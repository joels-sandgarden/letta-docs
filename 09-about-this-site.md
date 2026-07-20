# About this site

This field guide maps the current `letta-code` harness and the `letta-agent-sdk` client to the code that shapes them. It gives readers a mental model of the v2 Letta system: what each part does, how the parts fit together, and where the behavior lives in the repositories.

The official documentation at https://docs.letta.com covers quickstarts, feature how-tos, configuration, and protocol or API reference. This guide complements that material by staying at the system level. It keeps a reader close to the implementation without carrying old assumptions from earlier versions.

## Who this is for

This guide is for engineers adopting Letta, embedding it into another product, extending the harness, or contributing to the SDK or app-server boundary. It also serves readers who remember the MemGPT-era v1 server and need the current v2 model without treating that older design as current architecture.

## How it was made

Doc Holiday generated this guide by directly exploring the current `letta-code` and `letta-agent-sdk` repositories. The write-up follows the live code paths that explain the platform framing, including `src/agent/memory-filesystem.ts` (`resolveScopedMemoryDir`, `applyMemfsFlags`), `src/queue/turn-queue-runtime.ts` (`mergeQueuedTurnInput`), `src/channels/registry.ts` (`ChannelRegistry`, `initializeChannels`, `completePairing`), and `src/mods/mod-engine.ts` (`createModEngine`, `loadLocalMods`). It also traces the SDK and app-server boundary through `letta-agent-sdk/src/protocol.ts`, `letta-agent-sdk/src/session.ts`, `letta-agent-sdk/src/app-server-session.ts`, `src/app-server-client.ts` (`AppServerClient`, `runTurn`), and `src/websocket/listen-client.ts` (`startListenerClient`).

## What the guide emphasizes

The guide keeps the mental model on the current v2 path: memory helpers decide where a scoped memory directory lives, turn input merges before it reaches the runtime, channel registry code buffers ingress until the listener is ready, and mods load through a dedicated engine that tracks ownership and diagnostics. The SDK files define the protocol and session boundary, so the public client stays separate from the app-server transport.

## Scope and honesty

This page and the rest of the field guide capture a dated snapshot of the v2 codebase. Some paths still change or remain partial, so the guide treats them as observed facts at the time of generation rather than permanent architecture. The official docs remain authoritative for user-facing guidance, and corrections are welcome when the code changes or this snapshot drifts.

Generation stamp: latest observed `letta-code` main commit `5c236cb` dated `2026-07-20`.
Contact / repo note: [CONTACT_OR_REPO_LINK]

## Table of contents

- [the map of the v2 system](./00-the-big-picture.md)
- [end-to-end trace of one turn: message → queue → context assembly → LLM loop → client-side tools → streamed reply](./01-anatomy-of-a-turn.md)
- [agents vs conversations, turn serialization, steering mid-run, and self-scheduling](./02-conversations-queues-and-interrupts.md)
- [the two-layer memory model: persona/human blocks and the git-tracked MemFS](./03-memory-blocks-and-the-memory-filesystem.md)
- [sleep-time compute: how agents rewrite their own memory and learn skills in the background](./04-dreaming-and-reflection.md)
- [the three extension mechanisms and why each exists](./05-skills-subagents-and-mods.md)
- [client-side tool execution: hooks, approvals, permission modes, sandboxing](./06-tools-permissions-and-sandboxing.md)
- [one agent on many surfaces: the channel plugin architecture](./07-channels.md)
- [the websocket protocol seam and embedding agents programmatically](./08-the-app-server-and-the-sdk.md)

## Where to look in the code

- `README.md` in `letta-code` explains the public harness framing and current product model.
- `src/agent/memory-filesystem.ts`, `src/queue/turn-queue-runtime.ts`, and `src/channels/registry.ts` in `letta-code` show memory directory handling, turn merging, routing, and pairing.
- `src/mods/mod-engine.ts` in `letta-code` describes local mod loading, diagnostics, and lifecycle.
- `src/app-server-client.ts` and `src/websocket/listen-client.ts` in `letta-code` show app-server turn flow and the listener entrypoint.
- `README.md`, `src/protocol.ts`, `src/session.ts`, and `src/app-server-session.ts` in `letta-agent-sdk` define the SDK surface, session flow, and the app-server boundary.