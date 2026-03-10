# Agentic AI Implementation Plan

## Goal

Build a project with the same core architecture and capabilities as OpenCode so you can learn how a real agentic AI system is designed, implemented, and extended.

This plan is optimized for learning, not speed.

You should build the system in layers:

1. Local runtime and server
2. Session and persistence model
3. Agent loop and tool execution
4. CLI/TUI and frontend clients
5. Provider abstraction
6. Permissions and safety
7. Advanced workflows like subtasks, compaction, sharing, plugins, and remote control

The intent is to recreate the same major ideas as OpenCode:

- local-first agent runtime
- embedded HTTP server
- CLI and UI clients talking to the same backend
- session/message/part data model
- tool calling loop
- model provider abstraction
- permission system
- event bus and streaming updates
- plugin architecture
- multi-agent and subtask workflows

## Expected Outcome

By the end, you should have:

- a working local coding agent
- a headless server mode
- a CLI client
- a basic web or desktop client
- file and shell tools
- persisted sessions and resumable execution
- provider-agnostic model integration
- permission prompts and safety controls
- a strong understanding of agentic AI architecture

## Constraints

- Use TypeScript.
- Prefer Bun if you want something close to OpenCode.
- Use SQLite for local persistence.
- Keep the core runtime API-first from day one.
- Do not start with a frontend-heavy build.
- Do not start with multi-agent features.
- Do not start with plugins.

## Suggested Stack

- Runtime: Bun + TypeScript
- API server: Hono
- Validation: Zod
- Database: SQLite + Drizzle ORM
- CLI: yargs
- Web client: Solid or React
- Desktop: Tauri later
- AI SDK layer: Vercel AI SDK or direct provider adapters

---

## Phase 0: Preparation

### Day 1: Define the product boundary

Write a short design doc that answers:

- What exactly your agent can do
- What it is not allowed to do yet
- Which user interface ships first
- Whether you support local-only use or remote attach
- Which model providers you support first

Deliverables:

- `docs/vision.md`
- `docs/non-goals.md`
- `docs/use-cases.md`

What to learn:

- Agentic AI is mostly systems design, not prompt writing.
- The biggest mistake is unclear scope.

### Day 2: Study the target architecture

Read through OpenCode-style subsystems and map them into your own design:

- CLI entrypoint
- embedded server
- sessions
- tools
- providers
- permissions
- UI clients
- plugin hooks

Deliverables:

- `docs/architecture.md`
- a text diagram of data flow

What to learn:

- The core product is the runtime engine, not the UI.

### Day 3: Create the monorepo skeleton

Set up the repo as a workspace with packages like:

- `packages/runtime`
- `packages/sdk`
- `packages/app`
- `packages/desktop`
- `packages/plugin`
- `packages/util`

Deliverables:

- workspace config
- package manifests
- shared tsconfig
- root scripts

What to learn:

- Good package boundaries matter early.

---

## Phase 1: Core Runtime

### Day 4: Build the local server shell

Create a Hono server with:

- health route
- config route
- session route placeholder
- event route placeholder

Deliverables:

- `packages/runtime/src/server/server.ts`
- `packages/runtime/src/server/routes/*`

What to learn:

- OpenCode’s runtime is server-centered.

### Day 5: Build the CLI entrypoint

Create a CLI that:

- starts the local runtime
- calls the local server through an SDK client
- supports `run` and `serve`

Deliverables:

- `packages/runtime/src/index.ts`
- `packages/runtime/src/cli/cmd/run.ts`
- `packages/runtime/src/cli/cmd/serve.ts`

What to learn:

- The CLI should be a client of the runtime, not the runtime itself.

### Day 6: Add instance bootstrapping

Implement project-scoped initialization:

- current working directory detection
- instance context
- lifecycle setup and teardown
- per-project state container

Deliverables:

- `project/instance.ts`
- `project/bootstrap.ts`

What to learn:

- Agent behavior depends heavily on project context.

---

## Phase 2: Persistence and Session Model

### Day 7: Design the database schema

Create tables for:

- projects
- sessions
- messages
- parts
- permissions
- config or metadata if needed

Important: copy the OpenCode idea of `session -> messages -> typed parts`.

Deliverables:

- Drizzle schema files
- migration scripts

What to learn:

- This data model is one of the most important architectural decisions.

### Day 8: Implement the session service

Support:

- create session
- list sessions
- get session
- update title
- archive session
- fork session later

Deliverables:

- `session/index.ts`
- `server/routes/session.ts`

What to learn:

- The agent needs durable state, not just in-memory chat history.

### Day 9: Implement messages and parts

Represent assistant output as typed parts:

- text
- reasoning
- tool
- step-start
- step-finish
- patch
- file attachment

Deliverables:

- `session/message.ts`
- `session/message-v2.ts`

What to learn:

- Plain strings are not enough for an agent runtime.

### Day 10: Add event bus and streaming

Create an internal event system and stream updates over SSE.

Events should include:

- session created
- session updated
- message updated
- message part updated
- session status changed
- permission asked

Deliverables:

- `bus/*`
- `/event` SSE route

What to learn:

- Agent UX depends on real-time state propagation.

---

## Phase 3: LLM Execution Loop

### Day 11: Implement provider abstraction v1

Build a provider interface that supports:

- parse provider/model string
- list configured providers
- resolve model
- stream model output

Support one provider first.

Deliverables:

- `provider/provider.ts`
- `provider/types.ts`

What to learn:

- Do not couple the agent loop to one vendor SDK.

### Day 12: Implement the basic prompt loop

Flow:

1. user message enters session
2. model is selected
3. system prompt is built
4. tools are passed to model
5. streamed output is persisted
6. loop stops or continues after tool calls

Deliverables:

- `session/prompt.ts`

What to learn:

- This is the heart of the whole system.

### Day 13: Build the stream processor

Consume provider stream events and convert them into persisted parts.

Handle:

- text start/delta/end
- reasoning start/delta/end
- tool call start
- tool result
- error
- finish

Deliverables:

- `session/processor.ts`

What to learn:

- A good processor isolates provider-specific streaming details from the rest of the app.

### Day 14: Add retry and error handling

Implement:

- retryable provider errors
- aborted session state
- model not found handling
- fatal session errors

Deliverables:

- `session/retry.ts`
- better error types

What to learn:

- Reliability work is core product work.

---

## Phase 4: Tooling System

### Day 15: Build the tool contract

Each tool should define:

- id
- description
- Zod schema
- executor
- optional validation error formatter

Deliverables:

- `tool/tool.ts`
- `tool/registry.ts`

What to learn:

- The tool system is the execution surface of the agent.

### Day 16: Implement read-only tools

Start with:

- list files
- read file
- glob
- grep

Deliverables:

- `tool/list.ts`
- `tool/read.ts`
- `tool/glob.ts`
- `tool/grep.ts`

What to learn:

- Agent usefulness starts with environment visibility.

### Day 17: Implement write tools

Add:

- write file
- edit file
- maybe patch apply

Deliverables:

- `tool/write.ts`
- `tool/edit.ts`

What to learn:

- Structured editing is safer than raw shell writes.

### Day 18: Implement shell tool

Add:

- command execution
- cwd handling
- output streaming or buffered output
- process cancellation

Deliverables:

- `tool/bash.ts`
- `shell/shell.ts`

What to learn:

- Shell access is powerful and dangerous. Design it carefully.

### Day 19: Add tool result rendering in CLI

Make the CLI show:

- read/list actions
- file edits
- shell output
- tool failures

Deliverables:

- richer `run` output formatting

What to learn:

- Agent transparency is as important as capability.

---

## Phase 5: Permissions and Safety

### Day 20: Build permission rules

Support:

- allow
- deny
- ask

By tool and pattern:

- file path
- external directory
- shell command pattern

Deliverables:

- `permission/next.ts`

What to learn:

- A real local agent must have a policy engine.

### Day 21: Add permission request flow

When a tool needs approval:

- publish a permission event
- block until user replies
- continue or reject

Deliverables:

- `/permission` API
- permission reply path in CLI

What to learn:

- Human-in-the-loop control is central to agent systems.

### Day 22: Add safe defaults and agent modes

Create modes similar to:

- `build`
- `plan`
- `general`

Each mode should bundle:

- prompt style
- permissions
- optional model override

Deliverables:

- `agent/agent.ts`

What to learn:

- Modes are policy bundles, not just different prompts.

---

## Phase 6: Agent Layer

### Day 23: Implement system prompt assembly

Build prompt layers:

- global instructions
- agent prompt
- project context
- command context
- safety reminders

Deliverables:

- `session/system.ts`
- `session/instruction.ts`

What to learn:

- Good agent behavior comes from structured prompt assembly, not one giant prompt blob.

### Day 24: Add command templates

Implement reusable commands like:

- init
- explain
- fix
- summarize

Deliverables:

- `command/*`
- command route and CLI support

What to learn:

- Commands are productized workflows on top of the same session engine.

### Day 25: Add title generation and summaries

Generate:

- session titles
- quick summaries
- change summaries

Deliverables:

- `session/summary.ts`
- `session/title.ts` or helper within prompt flow

What to learn:

- Small metadata features greatly improve UX.

---

## Phase 7: File Awareness and Code Context

### Day 26: Implement snapshot and diff tracking

Capture filesystem state before and after tool execution.

Store:

- changed files
- patch summary
- revert metadata

Deliverables:

- `snapshot/*`

What to learn:

- Agents need auditable side effects.

### Day 27: Implement revert

Allow reverting a message’s file changes.

Deliverables:

- `session/revert.ts`

What to learn:

- Reversibility is one of the best safety and learning features.

### Day 28: Add project and VCS awareness

Detect:

- git branch
- worktree root
- project name
- repo state

Deliverables:

- `project/project.ts`
- `project/vcs.ts`

What to learn:

- Coding agents need repository awareness, not just filesystem access.

---

## Phase 8: Frontend Clients

### Day 29: Build SDK generation path

Expose OpenAPI and generate a TypeScript SDK from the runtime server.

Deliverables:

- `packages/sdk`
- OpenAPI generation script

What to learn:

- Shared contracts reduce drift across clients.

### Day 30: Build a minimal web client

Support:

- list sessions
- open session
- stream assistant updates
- submit prompt
- show tool usage

Deliverables:

- `packages/app`

What to learn:

- The frontend should render the event model, not invent its own state machine.

### Day 31: Add desktop wrapper

Wrap the web client with Tauri.

Add:

- native notifications
- deep links
- local process integration

Deliverables:

- `packages/desktop`

What to learn:

- Keep the UI reusable and platform-specific code thin.

---

## Phase 9: Advanced Runtime Features

### Day 32: Add context compaction

When sessions get too large:

- summarize prior history
- preserve key decisions
- continue execution with reduced context

Deliverables:

- `session/compaction.ts`

What to learn:

- Long-running agents require memory management.

### Day 33: Add subtasks and task tool

Enable the main agent to spawn a focused sub-agent task.

Deliverables:

- `tool/task.ts`
- subtask part type

What to learn:

- Multi-agent systems are usually controlled orchestration, not autonomous swarms.

### Day 34: Add structured output mode

Support user requests for strict JSON/schema output.

Deliverables:

- structured output tool or equivalent finalization path

What to learn:

- Agents often need machine-readable completion modes.

### Day 35: Add async session execution

Support:

- fire-and-forget prompts
- reconnecting clients
- resuming streamed updates

Deliverables:

- async prompt endpoint
- client resume logic

What to learn:

- Decoupled execution is necessary for robust clients.

---

## Phase 10: Extensibility

### Day 36: Build plugin hooks

Add hook points for:

- config load
- auth
- shell env
- command before/after
- tool before/after
- text transform
- event observer

Deliverables:

- `plugin/index.ts`
- `packages/plugin`

What to learn:

- Good agent systems separate core runtime from extensions.

### Day 37: Add dynamic plugin loading

Load plugins from:

- config
- local path
- package manager

Deliverables:

- plugin installation/loading flow

What to learn:

- Runtime extension has serious trust and versioning implications.

### Day 38: Add custom providers and custom agents from config

Support:

- provider definitions in config
- agent definitions in config
- user overrides

Deliverables:

- config merge layer

What to learn:

- A lot of agent power comes from configuration architecture.

---

## Phase 11: Integrations

### Day 39: Add LSP support

Integrate language servers for:

- diagnostics
- symbol lookup
- maybe code navigation helpers

Deliverables:

- `lsp/*`

What to learn:

- Coding agents improve dramatically with language-aware context.

### Day 40: Add MCP support

Let the runtime talk to external tool servers through MCP.

Deliverables:

- `mcp/*`
- MCP routes

What to learn:

- Modern agents increasingly rely on tool interoperability standards.

### Day 41: Add web search or fetch tools

Support:

- web fetch
- optional search tool

Deliverables:

- `tool/webfetch.ts`
- `tool/websearch.ts`

What to learn:

- External knowledge tools should be isolated from local code tools.

---

## Phase 12: Product Polish

### Day 42: Add auth and model config UX

Support:

- storing provider credentials
- listing models
- selecting default model

Deliverables:

- `auth/*`
- `config/*`
- `provider routes`

What to learn:

- Provider UX is product-critical in multi-model tools.

### Day 43: Add logs, metrics, and debugging paths

Support:

- service logs
- request logs
- session debug dump
- status inspection

Deliverables:

- `util/log.ts`
- debug CLI commands

What to learn:

- You cannot debug an agent platform without strong observability.

### Day 44: Add share/export/import

Support:

- shareable session link
- session export
- session import

Deliverables:

- `share/*`
- export/import commands

What to learn:

- Conversation portability is useful for support and learning.

### Day 45: Final architecture review

Review the whole system:

- package boundaries
- hot spots
- giant files
- testing gaps
- plugin trust model
- failure modes

Deliverables:

- `docs/retrospective.md`
- `docs/next-steps.md`

What to learn:

- The review phase is where architecture lessons become explicit.

---

## What “Same as OpenCode” Actually Means

If you truly want something very close to OpenCode, you need all of these:

- CLI entrypoint with `run` and `serve`
- embedded Hono server
- local and remote attach mode
- SQLite-backed sessions
- session/message/part event model
- SSE streaming
- provider-agnostic model layer
- agent mode and permission rules
- file tools and shell tool
- snapshots, diffs, revert
- command templates
- compaction and summaries
- plugin loading
- SDK generated from server contract
- web client and desktop wrapper

Do not try to ship all of that in week 1.
Build it in the order above.

---

## Recommended Weekly Rhythm

For each day:

1. Read the subsystem you are about to implement
2. Write a short design note before coding
3. Build the smallest correct version
4. Test it manually end to end
5. Write down what you learned

Keep a learning journal:

- what architectural choice you made
- what tradeoff you accepted
- what broke
- what you would redesign later

This is how you turn the project into agentic AI education instead of just cloning features.

---

## Testing Checklist By Milestone

After Phase 3:

- create session
- send prompt
- receive streamed text
- persist history

After Phase 4:

- tool calls execute correctly
- outputs are persisted
- failures are surfaced

After Phase 5:

- dangerous tools require approval
- denied tools stop or redirect flow correctly

After Phase 8:

- CLI and web client both work against the same runtime

After Phase 10:

- plugin hook execution does not break core runtime

After Phase 12:

- long sessions still behave correctly
- reconnect works
- revert works
- remote/headless mode works

---

## Final Advice

If your purpose is learning agentic AI, focus on these five concepts more than anything else:

1. Agent loop orchestration
2. Tool execution and permissioning
3. Session persistence and resumability
4. Provider abstraction
5. Event-driven client updates

Those five ideas are the real core.

Everything else is product expansion.
