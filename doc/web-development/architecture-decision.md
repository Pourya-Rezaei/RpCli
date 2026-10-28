# Web Interface Architecture Decision

ID: `WEB-001`; date: `2026-10-28`; status: overall direction accepted, technical details proposed.

## Problem and Goal

The user wants to work with the same DeepSeek service and project from a browser without depending on the terminal chat experience. The goal is to add a user-friendly interface, not replace the model service or rewrite the agent.

## Confirmed Decisions

| ID | Decision | Consequence |
| --- | --- | --- |
| D-01 | Preserve the current DeepSeek web connection method | The existing web token, PoW, and transport remain the foundation |
| D-02 | Preserve existing CLI paths | terminal, prompt, commit, resume, and serve must not regress |
| D-03 | Add an assistant-ui-based interface | The library owns the conversation experience, not file or shell execution |
| D-04 | Separate the interface and integration from the core | Keep upstream and library changes as localized as possible |
| D-05 | Preserve the ability to receive updates from both projects | Do not maintain an independent library fork or extensively copy core code |

## Proposed Architecture

```text
Browser: assistant-ui + React
              |
       Web runtime adapter
              |
       Local backend bridge
              |
 Existing agent / tools / DeepSeek transport

Existing CLI ----------------> Existing agent / transport
```

The web interface displays messages and execution status. The local bridge coordinates session identity, event streaming, cancellation, and approval. The core continues to enforce execution decisions and system access. The browser bundle must not contain Node modules, the DeepSeek token, or shell execution.

The proposed frontend is React + Vite within an independent build boundary; this choice still requires packaging review. In the installed version, assets are prebuilt and shipped with the application; running each session must not require a development server or package downloads. Text generation still requires a DeepSeek connection.

For initial chat, `useLocalRuntime` and `ChatModelAdapter` are proposed options. If ownership of complex agent state moves to an independent store, reevaluate `useExternalStoreRuntime` or an appropriate transport; do not install multiple parallel runtimes without a need. Verify the exact version and API against that version's documentation during implementation.

## Why Is Connecting Directly to the Current Server Insufficient?

In [session.ts](../../source/server/session.ts), HTTP requests run with `toolsEnabled: false`. [request.ts](../../source/server/request.ts) converts only the last message into a prompt. [apiClient.ts](../../source/core/apiClient.ts) has shared session and generation state. Simple chat can therefore be connected, but this path does not establish parity with the CLI assistant or support multiple concurrent sessions.

For agent capabilities, the bridge must consume the existing [getAIResponse](../../source/actions/agent.ts) and [executor](../../source/tools/execute.ts); do not write an independent tool loop in the browser. Limited extraction of Ink-dependent logic may be necessary. "Preserve the core" does not mean absolutely prohibiting compatibility changes, but each change must be small, justified, and tested.

## Options Compared

| Option | Assessment |
| --- | --- |
| Rewrite the entire application in a web framework | More upstream conflicts and behavioral risk; not recommended |
| Connect chat only to the existing completion endpoint | Suitable for a limited prototype; not a replacement for the full coding agent |
| Add an interface with a narrow bridge | Selected proposal; core reuse with independent interface control |
| Build a desktop application from the outset | More packaging and maintenance; outside the first release |

## Open Decisions Before Implementation

- Web entry command: a separate command, an opt-in option, or a change to the `rc` default? The current condition is to preserve existing behavior.
- Where exactly should the frontend and bridge live, and how should assets be published? Proposed paths are not yet contracts.
- How should token setup work from the web? Reusing existing configuration without exposing secrets is the initial baseline; a web form needs a separate contract.
- How should the mapping between UI identifiers, remote sessions, and history be versioned and stored?

Initially, allow one active run. Multiple concurrent remote sessions, multi-user deployment, real uploads, history editing, branching, and regeneration are out of scope until correct contracts exist. These limits must be reflected in UI controls, not just documentation.

Review sources: [assistant-ui repository](https://github.com/assistant-ui/assistant-ui), [LocalRuntime](https://www.assistant-ui.com/docs/runtimes/custom/local-runtime), [ExternalStoreRuntime](https://www.assistant-ui.com/docs/runtimes/custom/external-store). This decision is not a tested working example. The [development rules](development-rules.md) and [delivery stages](delivery-plan.md) complement it.