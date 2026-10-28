# Open WebUI Integration Decision

Decision: `WEB-002`. Date: `2026-10-28`. Baseline: RpCli `0.10.1`.
Status: direction accepted; implementation and runtime compatibility are not verified.

## Objective

Use the existing agent through Open WebUI instead of conducting everyday conversations in the terminal. Keep the terminal available. Do not build a custom frontend or replace the current DeepSeek connection. The earlier unimplemented custom-interface design is replaced by this decision.

## Accepted Boundaries

| ID | Decision | Consequence |
| --- | --- | --- |
| D-01 | Retain the DeepSeek web-session connection | Keep web tokens, PoW, transport, and current model interaction |
| D-02 | Preserve existing entry points | Terminal, prompt, commit, resume, and serve must not regress |
| D-03 | Run Open WebUI as a separate application | Do not import its frontend into RpCli or build replacement components |
| D-04 | Prefer configuration over code | Add a narrow adapter only for demonstrated integration gaps |
| D-05 | Deliver chat before file tools | Each later operation requires its own tests and approval gate |
| D-06 | Add no state-management package now | Use existing state ownership; address backend correctness explicitly |
| D-07 | Keep both upstream update paths open | Avoid Open WebUI source modifications and duplicated RpCli internals |

## Target Topology

```text
Browser
   |
Open WebUI application and its backend
   | OpenAI-compatible HTTP / supported integration extension
RpCli local service
   |
Existing agent and DeepSeek transport

Existing terminal / prompt / commit / resume remain available
```

Start with `rc serve` and an independently installed Open WebUI release. Do not change bare `rc`, introduce a launcher, or bundle another frontend during milestone 1. Host, port, installation method, and authentication must be recorded for the actual environment before execution. No service is authorized for public exposure.

Open WebUI's built-in direct DeepSeek provider is not the selected connection: that would bypass RpCli. Open WebUI must call RpCli, which retains the existing upstream credential and transport.

## Verified Existing Constraints

These observations are source-derived, not proof of successful interoperability:

- [server/index.ts](../../source/server/index.ts) exposes model discovery and chat completion endpoints. Similar endpoint names do not establish full protocol compatibility.
- [server/session.ts](../../source/server/session.ts), `runCompletion`, disables local tools with `toolsEnabled: false`. A working chat connection does not make the coding agent's file tools available.
- [server/request.ts](../../source/server/request.ts), `buildCompletionPrompt`, uses only the last message. Standard history replay, edit, regeneration, and branching semantics are not implemented by that conversion.
- [core/apiClient.ts](../../source/core/apiClient.ts) holds active session, parent message, and generation state at module scope. The server queue serializes requests but does not isolate conversations.
- Open WebUI can issue auxiliary model requests, including title generation. Sending them to the same stateful backend may alter the active conversation. Disable them for the initial experiment or route them separately through a verified mechanism.

## State Ownership and Package Decision

| Owner | Existing state | Integration rule |
| --- | --- | --- |
| RpCli terminal UI | `useState` and `useRef` for messages, streaming, input, mode, and lifecycle | Leave the terminal implementation intact |
| RpCli API client | Module-level remote session, parent ID, abort controller, and response IDs | Do not mistake shared variables for per-chat isolation |
| RpCli HTTP service | Completion queue, initialized token, and tools cache | Map ownership explicitly if an adapter is required |
| Local transcript storage | Persisted interactive messages | Not automatically Open WebUI history or a web session store |
| Open WebUI | Its own UI state, conversation records, users, and storage | Use its supported contracts; do not maintain a second custom frontend store |

Evidence: [InteractiveChatView.tsx](../../source/components/InteractiveChatView.tsx), [useChatSession.ts](../../source/components/chat/useChatSession.ts), [apiClient.ts](../../source/core/apiClient.ts), [session.ts](../../source/server/session.ts), [SessionTranscript.ts](../../source/core/SessionTranscript.ts), and [package.json](../../package.json).

RpCli declares no direct Redux or Zustand dependency. That does not imply that every transitive dependency or Open WebUI has no internal state mechanism. No claim about their complete dependency graphs is needed for this decision.

Adding a frontend state package would not repair shared backend sessions, cancellation ownership, or approval authorization. If integration needs a mapping, define conversation/run ownership and persistence first, and use the simplest suitable backend structure. A new dependency requires a concrete unmet requirement and separate review.

## Proposed Integration Path

First evaluate a configuration-only OpenAI-compatible connection for one controlled conversation. Then test conversation correctness and determine whether a supported Open WebUI Pipe or a narrow external adapter is required. Do not assume an extension receives a stable chat identity or that the custom RpCli session response header is automatically reused: inspect and test the chosen version's contract.

Before adding tools, choose one orchestration owner: RpCli's existing agent and executor. Do not silently replace them with Open WebUI's own tool loop or an unrelated terminal service. A file-tool approval bridge must reach the existing executor; this is integration work, not a new agent implementation.

## Open Decisions

- Which pinned Open WebUI release and installation method will the owner use?
- How will the local RpCli endpoint be protected, and how will both processes reach it without public binding?
- Which supported extension can associate Open WebUI chat identities with RpCli sessions without changing legacy API behavior?
- How will cancellation, disconnects, tool approvals, and local/remote history ownership be represented?

If these cannot be resolved with small additive changes, stop and present the incompatibility. Do not broaden scope or promise that no integration code will be necessary.

## Sources and Scope

[Open WebUI repository](https://github.com/open-webui/open-webui), [OpenAI-compatible connection guide](https://docs.openwebui.com/getting-started/quick-start/connect-a-provider/starting-with-openai-compatible), and [license](https://github.com/open-webui/open-webui/blob/main/LICENSE). Recheck version-specific documentation during implementation. See [development rules](development-rules.md), [delivery plan](delivery-plan.md), and [update policy](update-policy.md).