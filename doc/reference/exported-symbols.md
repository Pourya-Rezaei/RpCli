# Important Interfaces and Symbols

This page locates architectural integration points rather than listing every private variable. Version `0.10.1`, reviewed `2026-10-28`; evidence was derived from code.

## Public API Status

[package.json](../../package.json) defines the executable `rc` and sets files to dist; no `exports` contract is defined for a stable SDK. A TypeScript function export does not guarantee compatibility for external consumers. Direct imports of internal paths are not a documented public package contract.

## Agent and API Client

| Module | Interface | Responsibility and Qualification |
| --- | --- | --- |
| [agent.ts](../../source/actions/agent.ts) | `ChatMode` | normal, yolo, plan |
| Same module | `AIResponseOptions` | Token, prompt, callbacks, mode, and model/tool options |
| Same module | `getChatSystemPrompt` | Prepare the agent context prompt |
| Same module | `getAIResponse` | New generation, tool loop, and final result |
| [apiClient.ts](../../source/core/apiClient.ts) | `SendMessageOptions`, default `sendMessage` | Upstream sending, PoW, and advancing the parent |
| Same module | `beginGeneration`, `isGenerationStopped`, `stopCurrentGeneration` | Control a generation and its stop flag |
| Same module | `resetChatSession`, `getCurrentSessionId`, `setCurrentSessionId` | Shared module-level state |
| Same module | `chatSessionExists` | Check for a session on the service; network-dependent |

API-client state does not belong to an independent chat-instance class. Creating multiple callers within one process does not itself provide isolation. Server, CLI, and transcript models should not be treated as a single data type.

## Lower-Level Transport

Export index: [core-lib/index.ts](../../core-lib/index.ts).

| Export | Owning Module | Responsibility |
| --- | --- | --- |
| `chat`, `ChatResult`, `ChatStreamChunk` | [Chat.ts](../../core-lib/Chat.ts) | Completion, SSE, metadata, and response continuation |
| `continueChat` | [ContinueChat.ts](../../core-lib/ContinueChat.ts) | Continuation request and stream body |
| `stopStream` | [StopStream.ts](../../core-lib/StopStream.ts) | Request remote response cancellation |
| `createSessions` | [CreateSessions.ts](../../core-lib/CreateSessions.ts) | Create a session |
| `chatSessions`, `ChatSession` | [ChatSessions.ts](../../core-lib/ChatSessions.ts) | List account sessions |
| `deleteSession` | [DeleteSession.ts](../../core-lib/DeleteSession.ts) | Send a deletion request |
| `createPowChallenge` | [CreatePowChallenge.ts](../../core-lib/CreatePowChallenge.ts) | Obtain a challenge before completion |
| `InvalidTokenError`, `assertValidTokenResponse`, `isInvalidTokenError` | [InvalidTokenError.ts](../../core-lib/InvalidTokenError.ts) | Classify credential errors |

These interfaces depend on the DeepSeek web API, not the official key-based API. See [DeepSeek Transport](../explanation/deepseek-transport.md).

## Tools and Commands

| Module | Important Symbols | Contract |
| --- | --- | --- |
| [tools/types.ts](../../source/tools/types.ts) | `Tool`, `ToolCall`, `ToolResult`, `ToolMode`, `ToolConfirmationDetails`, `ConfirmationHandler` | Description, input, output, and confirmation |
| [registry.ts](../../source/tools/registry.ts) | `tools`, `getToolDescriptions` | Registry and tool text for the prompt |
| [execute.ts](../../source/tools/execute.ts) | `executeTool`, `executeToolCalls`, `toolIsMutating`, `toolRequiresConfirmation` | Dispatch and batch policy; policy is not guaranteed by single-tool execution |
| [parse.ts](../../source/tools/parse.ts) | `parseToolCalls` | Convert markup into calls |
| [arguments.ts](../../source/tools/arguments.ts) | `stringArgument`, `textArgument`, `positiveIntegerArgument` | Read and convert arguments |
| [paths.ts](../../source/tools/paths.ts) | `rootDirectory`, `safePath`, `safeTargetPath` | Control tool paths |
| [todo.ts](../../source/tools/todo.ts) | `todoTools`, `getTodoList` | In-memory task state |
| [commands/index.ts](../../source/commands/index.ts) | `SlashCommand`, `SlashCommandContext`, `slashCommands`, `resolveSlashCommand`, `formatSlashCommandHelp` | Dispatch interface commands |

[tools/index.ts](../../source/tools/index.ts) is the consumer-facing barrel. Preview logic is in [describe.ts](../../source/tools/describe.ts), display diffs in [diff.ts](../../source/tools/diff.ts), and elevation in [sudo.ts](../../source/tools/sudo.ts). Adding an export without registering it in the registry is insufficient to make a tool available to the model.

## Sessions, Storage, and Interface

| Module | Symbol/Family | Responsibility |
| --- | --- | --- |
| [TokenConfig.ts](../../source/core/TokenConfig.ts) | tokenConfigDirectory, tokenConfigPath, save/clear/get preferences | Home configuration file |
| [SessionTranscript.ts](../../source/core/SessionTranscript.ts) | `sessionTranscriptDirectory`, `loadSessionTranscript`, `saveSessionTranscript`, `deleteSessionTranscript` | Local transcript with ID validation |
| [useChatSession.ts](../../source/components/chat/useChatSession.ts) | `useChatSession` | Initialize, reset, switch, and clean up |
| [createLiveStream.ts](../../source/components/chat/createLiveStream.ts) | `createLiveStream` | Aggregate live UI updates |
| [useComposerKeys.ts](../../source/components/chat/useComposerKeys.ts) | `useComposerKeys` | Input and mode control |
| [chat/types.ts](../../source/components/chat/types.ts) | Message and interface-state types | Display contract, not an OpenAI payload |

Bidi, cursor, logging, and discovery tools are also internal utilities; file ownership details are in the [Coverage Map](source-map.md).

## Server

| Module | Main Exports/Interfaces |
| --- | --- |
| [server/index.ts](../../source/server/index.ts) | `startServer`, `ServerOptions` |
| [session.ts](../../source/server/session.ts) | `enqueueCompletion`, `runCompletion`, `getSessionTools`, `rememberSessionTools` |
| [request.ts](../../source/server/request.ts) | `generateCompletionId`, `resolveModel`, `resolveThinkingEnabled`, `buildCompletionPrompt`, `validateTools`, `withToolInstructions`, `parseClientToolCalls`, `toUsage` |
| [errors.ts](../../source/server/errors.ts) | `ApiError`, `errorBody`, `sendError`, `toApiError` |
| [types.ts](../../source/server/types.ts) | `ChatCompletionRequest`, `ChatMessage`, `FunctionTool`, `ToolChoice`, `ReasoningEffort`, `CompletionUsage`, and related types |

For external use, use the [HTTP Contract](http-api.md) instead of importing these functions. For in-repository development, [Extending Tools and Commands](../development/extending-tools-and-commands.md) and the [Prompt Reference](prompts.md) identify related integration points.