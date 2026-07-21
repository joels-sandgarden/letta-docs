# Skills, Subagents, and Mods

Extending an agent can mean giving it knowledge, giving it help via delegation, or changing what the harness itself can do. In the Letta agent harness, letta-code, those three moves stay separate on purpose: skills shape prompt-time knowledge, subagents shape delegation boundaries, and mods shape the host harness.

Skills are the lightest surface, and they answer a simple question: what procedure should the agent remember at prompt time? `discoverSkills()` walks four scopes in order of precedence — project, agent, global, bundled — and it resolves duplicate skill IDs by keeping the highest-priority source. The runtime selector in `skill-sources.ts` trims that set for `noSkills` or `noBundledSkills`, while `all` selects every source as the default. Each skill stays a recursive `SKILL.md` file, which keeps the reuse unit small and legible instead of turning the system into a general knowledge store.

The harness does not treat skills as memory blocks. `formatSkillsAsSystemReminder()` turns the discovered set into a system reminder, and `prependReminderPartsToContent()` folds reminder text into the prompt content that actually reaches the model. `buildSharedReminderParts()` supplies that reminder stream, while `injectQueuedSkillContent()` appends queued skill content as a trailing user message when a skill emits procedural text at runtime. That split matters: a skill can guide behavior without becoming durable memory, and it can still inject fresh instructions when the task demands it. The reminder path stays visible at turn time instead of disappearing into long-lived storage.

The agent-scoped filesystem at `~/.letta/agents/{agent-id}/memory/skills/` gives the cleanest version of that loop. A reflection pass writes a new skill there when a workflow repeats often enough to deserve procedural memory, which keeps the boundary sharp between recall and procedure. The memory filesystem page explains that split in more detail, and the reflection page shows when the harness promotes a repeated workflow into a reusable skill: [memory blocks and the memory filesystem](/03-memory-blocks-and-the-memory-filesystem.md) and [dreaming and reflection](/04-dreaming-and-reflection.md).

Subagents answer a different question: where should work run so the main context stays clean? `discoverSubagents()` and `getBuiltinSubagents()` assemble the available fleet from bundled Markdown files and from custom `.md` files under `.letta/agents/`. Built-ins ship with the product; custom subagents live beside a project and let local teams tune delegation without editing the harness itself. Custom subagents use Markdown frontmatter, so a project can define a local delegation shape without inventing a new control plane. `spawnSubagent()` and `manager.ts` then launch the child agent, collect the report, and route the result back through the stream and queue path that handles turns. That makes subagents a controlled side channel, not a second main agent.

The launch path narrows the child on purpose. `buildSubagentArgs()` constrains tools and preloads any declared skills. `composeSubagentChildEnv()` isolates environment and memory scope, and `resolveSubagentWorkingDirectory()` keeps memory-subagent runs pointed at the right filesystem root. That separation lets a child inherit just enough context to stay useful while still avoiding pollution of the parent working set. The built-in fleet follows roles, not a catalog: a general purpose worker, a forked background worker, recall and history analysis workers, initialization, memory work, and reflection. Some built-ins preload skills, and reflection can create or update skills when it finds a workflow that should survive beyond one conversation. For the interruption and queue model that makes those reports safe to consume, see [conversations, queues, and interrupts](/02-conversations-queues-and-interrupts.md).

Mods sit below skills and subagents. They change what the machine can do, not just what the agent knows or how it delegates work. `createModEngine()` loads local mods, manages activation and reloads, and keeps diagnostics attached to live registrations. `MOD_CAPABILITY_IDS` defines the surface that mods can ask for: tools, commands, providers, permissions, event groups, and `ui.panels`. `resolveModCapabilities()` keeps that surface explicit, and the default or disabled presets let the host widen or narrow the available surface. `registerModTool()` and `registerModPermission()` write into shared registries, which makes those capabilities visible across the host instead of keeping them trapped inside one mod file.

The architecture note makes the line clear: a mod is trusted local code, host APIs stay thin, and cleanup matters more than compatibility. The engine follows that idea closely. It registers only the capabilities the host allows, removes stale handles on reload or shutdown, and treats recovery as a first-class concern. A skill teaches an agent, a subagent does scoped work, and a mod changes the substrate under both. See the architecture note in `src/skills/builtin/creating-mods/references/architecture.md` and the core engine files under `src/mods/`.

Honesty note, mid-2026: inventories and edge details drift quickly, so the useful shape matters more than the exact list.

The practical decision stays simple:

- Need a repeatable procedure or domain habit — use a skill.
- Need scoped work that should not clutter the main context — use a subagent.
- Need a new capability, integration, permission surface, or UI hook — use a mod.
- Need both a capability and guidance for using it — pair a mod with a skill.
- Need reusable reflection on repeated behavior — let the reflection subagent write the skill.

Official docs live at https://docs.letta.com/letta-agent/skills, https://docs.letta.com/letta-agent/subagents, https://docs.letta.com/letta-agent/mods, and https://docs.letta.com/letta-agent/hooks.

## Where to look in the code

- `src/agent/skills.ts` — discovery, precedence, and system reminder formatting.
- `src/agent/skill-sources.ts` — source selection, `all`, and the `noSkills` / `noBundledSkills` filters.
- `src/agent/subagents/index.ts` — built-in and custom subagent discovery.
- `src/agent/subagents/manager.ts` — child launch, result collection, and retries.
- `src/agent/subagents/subagent-launcher.ts` — child environment and working directory isolation.
- `src/mods/mod-engine.ts` — mod loading, lifecycle, and registry orchestration.
- `src/mods/tool-registry.ts` and `src/mods/permission-registry.ts` — shared tool and permission registries.
- `src/mods/capabilities.ts` — the mod capability surface and presets.