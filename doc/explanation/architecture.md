# Architecture and Module Ownership

Review date: 2026-10-28 | Version: 0.10.1
This explanation is based on reading the code; runtime behavior has not been tested.

## Mental Model

RpCli is a terminal interface over DeepSeek web sessions, not a local model engine.
The agent coordinator converts the model's text output into tool requests and sends the results back to the same conversation.
The HTTP interface uses the same coordinator, but does not locally execute tools declared by the client.
Therefore, "model tools", "local file operations", and "HTTP client function calls" do not share a single execution boundary.

```text
cli -> Ink / app -> views -> getAIResponse -> sendMessage -> core-lib -> DeepSeek
cli -> startServer -> request / session / stream -> getAIResponse
getAIResponse -> parseToolCalls -> executeToolCalls -> local tools
```

## Entry and Interface Selection

[source/cli.tsx](../../source/cli.tsx) loads configuration and parses arguments with Meow.
The `serve` branch is selected before entering Ink; the server should not be treated as part of `App` routing.
In the terminal branch, the commit option, single-prompt input, and interactive chat have separate paths.
[source/app.tsx](../../source/app.tsx), through the `App` component, gates access to views on the presence of `DEEPSEEK_TOKEN`.
An invalid token also returns through this layer to the credential reset flow.
Command syntax details are in the [CLI reference](../reference/cli.md).

## Coordinator and Shared State

`getAIResponse` in [agent.ts](../../source/actions/agent.ts) owns the generation, tool parsing, and result feedback loop.
`getChatSystemPrompt` builds the conversation's initial text from prompts and project guidance.
`ChatMode` and `AIResponseOptions` define the contract between views and the agent.
This layer controls loop termination through a tool-round limit; it does not interpret SSE.
`sendMessage` in [apiClient.ts](../../source/core/apiClient.ts) manages the session, parent message, and PoW.
`beginGeneration` and `stopCurrentGeneration` in the same file own cancellation state.
The session ID and abort controller are stored at module scope, not in a separate object per client.
This choice matters when understanding the server queue and conversation isolation limitations.

## Ownership of All Transport Files

| File | Owning Symbols and Responsibility |
| --- | --- |
| [index.ts](../../core-lib/index.ts) | Re-exports transport operations and types; an export does not imply a stable SDK. |
| [Chat.ts](../../core-lib/Chat.ts) | `chat`, `ChatResult`, and `ChatStreamChunk`; completion, SSE reading, and continuation of incomplete responses. |
| [ContinueChat.ts](../../core-lib/ContinueChat.ts) | `continueChat`; requests continuation of an existing message. |
| [ChatSessions.ts](../../core-lib/ChatSessions.ts) | `chatSessions` and `ChatSession`; retrieves sessions and the current message ID. |
| [CreateSessions.ts](../../core-lib/CreateSessions.ts) | `createSessions`; creates a remote session ID. |
| [DeleteSession.ts](../../core-lib/DeleteSession.ts) | `deleteSession`; sends a deletion request, without guaranteeing confirmation of deletion. |
| [CreatePowChallenge.ts](../../core-lib/CreatePowChallenge.ts) | `createPowChallenge`; obtains a challenge to solve locally. |
| [StopStream.ts](../../core-lib/StopStream.ts) | `stopStream`; requests that the remote service stop a message. |
| [InvalidTokenError.ts](../../core-lib/InvalidTokenError.ts) | `InvalidTokenError`, `assertValidTokenResponse`, and `isInvalidTokenError`; detects credential errors. |

## Terminal Views and Components

| Component | Responsibility Boundary |
| --- | --- |
| [InteractiveChatView](../../source/components/InteractiveChatView.tsx) | Input, history, commands, model, work mode, and connecting the stream to the display. |
| [SinglePromptView](../../source/components/SinglePromptView.tsx) | One prompt, approval of its tools, and exit after the response. |
| [CommitView](../../source/components/CommitView.tsx) | Displays the proposed message and requests confirmation before running Git. |
| [TokenSetupView](../../source/components/TokenSetupView.tsx) | Collects and stores the web session token using masked input. |
| [MarkdownText](../../source/components/MarkdownText.tsx) | Converts a subset of Markdown into Ink components. |
| [FzfFilePicker](../../source/components/FzfFilePicker.tsx) | Selects paths using internal fuzzy matching, without running the external fzf program. |
| [SlashCommandPicker](../../source/components/SlashCommandPicker.tsx) | Suggests and selects slash commands. |
| [ToolConfirmation](../../source/components/ToolConfirmation.tsx) | Collects the user's decision about tool execution. |
| [Spinner](../../source/components/Spinner.tsx) | Waiting indicator, not a source of network state. |
| [RpCliLogo](../../source/components/RpCliLogo.tsx) | Displays the application's identity in history. |
| [MessageRow](../../source/components/chat/MessageRow.tsx) | Displays a message according to its role and content. |
| [ChatStatusBar](../../source/components/chat/ChatStatusBar.tsx) | Displays conversation state and active options. |
| [ModelConfirmation](../../source/components/chat/ModelConfirmation.tsx) | Interface decision for model selection. |
| [PlanConfirmation](../../source/components/chat/PlanConfirmation.tsx) | Decision to execute a plan after a plan-mode response. |

`useChatSession` in [useChatSession.ts](../../source/components/chat/useChatSession.ts) coordinates initialization, model switching, and cleanup of unused sessions.
`useComposerKeys` in [useComposerKeys.ts](../../source/components/chat/useComposerKeys.ts) separates keyboard decisions from the main view.
`createLiveStream` in [createLiveStream.ts](../../source/components/chat/createLiveStream.ts) batches display updates approximately every 50 milliseconds.
The `ChatMessage` type in [types.ts](../../source/components/chat/types.ts) is the interface message contract, not the DeepSeek response format.

## Tools, Prompts, and Persistent Data

[registry.ts](../../source/tools/registry.ts) declares tools, and `executeToolCalls` in [execute.ts](../../source/tools/execute.ts) applies execution-batch policy.
The `Tool` and `ToolCall` types in [types.ts](../../source/tools/types.ts) are local tool contracts.
`SystemPrompt` and `readAgentsMarkdown` in [prompts/index.ts](../../source/prompts/index.ts) use prompt files and only the `AGENTS.md` at the cwd root.
Prompt text expresses desired model behavior; it does not replace argument validation or path controls.
[TokenConfig.ts](../../source/core/TokenConfig.ts), [SessionTranscript.ts](../../source/core/SessionTranscript.ts), and [LogChat.ts](../../source/core/LogChat.ts) store configuration, conversation continuation transcripts, and optional event logs, respectively.
These three kinds of data are distinct from DeepSeek's remote history.

## Server Boundary and Further Reading

`startServer` in [server/index.ts](../../source/server/index.ts) registers routes; `runCompletion` and `enqueueCompletion` in [session.ts](../../source/server/session.ts) queue shared execution.
[request.ts](../../source/server/request.ts) interprets client input, and [stream.ts](../../source/server/stream.ts) handles the HTTP response format.
For event sequences, read [execution flows](execution-flows.md); for transport, read [DeepSeek](deepseek-transport.md); for trust boundaries, read [security and privacy](security-and-privacy.md).
More precise contracts appear in [exported symbols](../reference/exported-symbols.md), [local tools](../reference/local-tools.md), and [prompts](../reference/prompts.md).
The [documentation index](../README.md) lists reading paths.