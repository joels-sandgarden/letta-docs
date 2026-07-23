# The Big Picture

Letta v2 maps the current system that lets engineers work with agents across the `letta` CLI, the desktop app, `chat.letta.com`, messaging channels, and the Agent SDK. This page gives the current map for the system: it names the moving parts, shows where the harness ends and the backend begins, and keeps the MemGPT-era server in history.

The harness acts as a client. It can talk to three backend implementations: Letta Cloud / Constellation, a self-hosted App Server, and the experimental in-process local engine under `src/backend/local/`. The same turn model reaches all of them, but each backend owns a different slice of the runtime contract.

```mermaid
flowchart TB
  subgraph Surfaces["Surfaces"]
    direction LR
    CLI["`letta` CLI"]
    Desktop["Desktop app"]
    Chat["chat.letta.com"]
    Channels["Messaging channels"]
    SDK["Agent SDK"]
  end

  subgraph Harness["`letta-code` harness"]
    direction TB
    Ingress["Message enters the harness"]
    Queue["Queueing and turn setup"]
    Loop["Protocol V2 loop state"]
    Memory["Memory filesystem"]
    Extensions["Skills, subagents, mods"]
    ToolExec["Tool execution stays on the client machine"]
    Ingress --> Queue --> Loop --> Memory
    Loop --> Extensions
    Loop --> ToolExec
  end

  subgraph Backends["Backends"]
    direction LR
    Cloud["Letta Cloud / Constellation"]
    AppServer["Self-hosted App Server"]
    Local["Experimental local engine"]
  end

  CLI --> Ingress
  Desktop --> Ingress
  Chat --> Ingress
  Channels --> Ingress
  SDK --> Ingress

  Loop --> Cloud
  Loop --> AppServer
  Loop --> Local
```

## The turn loop at map altitude

Incoming messages land in a conversation-scoped queue, and the listener turns them into one turn at a time. Turn setup prepares the current agent and conversation, adds reminder context, builds the tool context, and carries forward any interrupted work that needs to resume. The lifecycle then moves through protocol states such as `SENDING_API_REQUEST`, `WAITING_FOR_API_RESPONSE`, `PROCESSING_API_RESPONSE`, `EXECUTING_CLIENT_SIDE_TOOL`, `WAITING_ON_APPROVAL`, and `WAITING_ON_INPUT` until the turn finishes.

`EXECUTING_CLIENT_SIDE_TOOL` marks the moment when the harness asks the user machine to run a tool. The model waits for that result; the tool does not move to the server. That split keeps local actions local while the protocol still records the turn in a single stream.

For the full turn map, see [Anatomy of a turn](./01-anatomy-of-a-turn.md) and [Conversations, queues, and interrupts](./02-conversations-queues-and-interrupts.md).

## Memory and identity

The memory filesystem lives on local disk as git repositories, and the small always in context memory blocks stay attached to the agent on every turn. Those blocks carry the stable identity of the agent: persona, human context, and other facts that should remain present without searching a larger store. The git-tracked memory tree carries the changing working memory, so identity stays continuous while the long-term memory keeps a revision history.

See [Memory blocks and the memory filesystem](./03-memory-blocks-and-the-memory-filesystem.md) for the memory model.

## Extension mechanisms and background improvement

Skills load reusable instructions and turn-specific tools into the current context.

Subagents let one agent hand work to another conversation with its own state and result stream.

Mods let local extensions register commands, tools, events, permissions, and panels without changing the core harness.

Reflection and dreaming keep background self-improvement running between turns so the system can rewrite memory or prepare future work.

For the deeper extension map, see [Dreaming and reflection](./04-dreaming-and-reflection.md) and [Skills, subagents, and mods](./05-skills-subagents-and-mods.md). Tool approval and sandbox rules live in [Tools, permissions, and sandboxing](./06-tools-permissions-and-sandboxing.md). Channel routing lives in [Channels](./07-channels.md).

## MemGPT / v1, historical only

In the 2024 MemGPT-era v1 design, the server owned tool execution, memory used embedding-backed folders and document search, and a heartbeat loop drove recurring work. The current v2 design moves tool execution to the client side, uses git-tracked MemFS and filesystem tools for memory, and replaces the heartbeat loop with crons, schedules, and post-turn reflection. In both systems, the memory blocks remain the continuity point for identity.

## Honesty note, mid 2026

As of mid-2026, the harness ships near daily, the local backend remains experimental, and the newest messaging channels are still young. This page captures the current map, not a promise that every edge will stay fixed.

## Where to look in the code

- `src/backend/backend.ts` and `src/backend/local/local-backend.ts` define the backend split and the experimental local engine.
- `src/types/protocol_v2.ts` defines the loop status model, the client-side tool phase, queue messages, and the memory and channel state surfaces.
- `src/websocket/listener/turn-setup.ts`, `src/websocket/listener/turn-lifecycle.ts`, and `src/websocket/listener/protocol-outbound.ts` show turn setup, lifecycle transitions, and outbound status updates.
- `src/agent/memory-filesystem.ts` and `src/queue/turn-queue-runtime.ts` show the memory directory layout and turn input merging.
- `src/tools/manager.ts`, `src/mods/mod-engine.ts`, and `src/channels/registry.ts` show tool assembly, local extensions, and channel ingress.
- `src/app-server-client.ts`, `letta-agent-sdk/src/protocol.ts`, `letta-agent-sdk/src/session.ts`, and `letta-agent-sdk/src/app-server-session.ts` define the app server transport and SDK session boundary.