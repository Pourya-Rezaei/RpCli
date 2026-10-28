# Execution Flows and Session Lifetime

Review date: 2026-10-28 | Version: 0.10.1
This text is derived from the code; no session, Git command, or network request was run to verify it.

## Three Distinct Concepts

A remote session is the thread of messages retained in DeepSeek.
A local transcript contains displayable messages for resuming the interactive interface.
The current generation is a cancellable execution that may include several generation requests and tool rounds.
The latter two concepts are owned by [SessionTranscript.ts](../../source/core/SessionTranscript.ts) and `beginGeneration` in [apiClient.ts](../../source/core/apiClient.ts), respectively.
Changing one does not necessarily delete the other two.

## Fresh Start

1. [cli.tsx](../../source/cli.tsx) reads configuration and arguments and selects the interface path.
2. `App` in [app.tsx](../../source/app.tsx) routes a missing token to `TokenSetupView`.
3. [InteractiveChatView.tsx](../../source/components/InteractiveChatView.tsx) loads the model preference and resets API state for a fresh conversation.
4. `ensureSession` and `startSession` in [useChatSession.ts](../../source/components/chat/useChatSession.ts) send the system prompt.
5. `sendMessage` finds or creates the session and attaches the message to the appropriate parent.

The initialization response is not displayed to the user as an ordinary response.
Sending user input waits for initialization to finish; displaying the screen alone does not mean the network is ready.
The hook's `sessionGeneration` counter separates an old initialization result from a new session.
If an old result arrives after a switch, a deletion request is sent for its session.

## Resuming a Conversation

With `resume <session-id>`, the interface path passes the requested ID to the hook.
`setCurrentSessionId` in [apiClient.ts](../../source/core/apiClient.ts) marks the parent message as unknown so it can be discovered on the next send.
The local transcript is loaded if present; the system prompt is not resent.
On the next send, `chatSessions` supplies the session's current message ID.
If the session is not found in the list, `sendMessage` can create a fresh session; resume does not guarantee an explicit failure for a nonexistent ID.
The absence of a local transcript is also distinct from the absence of remote history.
The storage format and IDs are explained in [configuration and storage](../reference/configuration-and-storage.md).

## A Turn and the Tool Loop

`getAIResponse` in [agent.ts](../../source/actions/agent.ts) owns this cycle:

```text
prompt -> sendMessage -> parse tool calls -> executeToolCalls
                       ^                         |
                       +------ tool_result ------+
```

Response text is checked for tool calls first; thinking text is checked only if the response contains no calls.
`parseToolCalls` in [parse.ts](../../source/tools/parse.ts) converts text into local calls.
A parse error consumes a round, and a correction message is sent to the model.
The execution result, including an error result, returns to the conversation inside a `tool_result` block.
At most 10 tool rounds are allowed; at the limit, the agent requests a summary without tools.
This number is not the SSE continuation limit or the total number of messages in a session.
Stream display is batched through `createLiveStream` in [createLiveStream.ts](../../source/components/chat/createLiveStream.ts), and tool markup remains hidden from the user.

## Mode and User Decisions

`useComposerKeys` in [useComposerKeys.ts](../../source/components/chat/useComposerKeys.ts) cycles through `normal`, `yolo`, and `plan` with TAB.
`executeToolCalls` in [execute.ts](../../source/tools/execute.ts) enforces the mode during tool execution.
In normal mode, a tool with `requiresConfirmation` needs a user decision; a missing callback does not mean approval.
In yolo mode, ordinary confirmations are skipped, but elevation detection still requires confirmation.
In plan mode, operations classified as modifications are blocked; adding and changing some in-memory todos remains possible.
`PlanPrompt` in [prompts/index.ts](../../source/prompts/index.ts) is supplementary behavioral guidance, not the mechanism enforcing this prohibition.
After a plan, `PlanConfirmation` in [PlanConfirmation.tsx](../../source/components/chat/PlanConfirmation.tsx) accepts Enter or `y` to execute and Esc or `n` to stay in plan mode.
Accepting the plan restores the previous non-plan mode and sends `ExecutePlanPrompt`; a plan is not a transaction that can be rolled back.

## Cancellation and Cleanup

While loading, Ctrl+C initiates local cancellation through `stopCurrentGeneration` and, if an ID is available, a remote stop request.
Session discovery, challenge retrieval, and synchronous PoW solving do not have the same AbortSignal coverage as the completion request.
Cancelling a response cannot undo file changes or commands that have already run.
Outside loading, Ctrl+C first clears existing input, after which the exit path comes into play.
`deleteUnusedSession` in [useChatSession.ts](../../source/components/chat/useChatSession.ts) performs best-effort cleanup of a fresh session with no user messages.
A used session is retained, and the interface can display a resume command on exit.
`resetConversation` deletes the previous transcript and resets API state; `/clear` does not necessarily delete the used remote session.
This command does not clear todos, nor does a fresh session alone completely replace the previous display history.

## Model Switching

Before the first user message, `switchModel` in [useChatSession.ts](../../source/components/chat/useChatSession.ts) does not need to create a replacement session.
After use, it obtains a summary with the previous model, without thinking or tools.
It then clears API state and sends the system prompt together with the summary to a fresh session for the target model.
The previously used session is not deleted; this is a summary transfer, not an exact copy of the history and all its context.
If the summary is empty, the initial prompt is built without a summary.

## Single Prompt, Commit, and Server

`SinglePromptView` in [SinglePromptView.tsx](../../source/components/SinglePromptView.tsx) sends system context first, followed by the prompt, and manages normal-mode tools.
At the end, it initiates a session deletion request without waiting for the result and exits; quiet only disables thinking display.
[commitMessage.ts](../../source/actions/commitMessage.ts) generates a message from the staged diff or `git diff HEAD` and awaits the deletion request before returning the text.
`CommitView` in [CommitView.tsx](../../source/components/CommitView.tsx) runs `git commit -F -` only after user approval.
The `-a` option changes the message-generation input; it neither stages changes by itself nor produces `git commit -a`.
On the server, `enqueueCompletion` and `runCompletion` in [session.ts](../../source/server/session.ts) share a queue and session state.
[request.ts](../../source/server/request.ts) uses only the last client message's text as input, rather than replaying the entire conversation array.
Client tools are returned as a response; this path is separate from local tool execution.
A disconnect disables output, but an upstream stop request is attempted after the completion returns.

## Related Reading

[Architecture](architecture.md), [DeepSeek transport](deepseek-transport.md), and [security](security-and-privacy.md) explain the reasons for these boundaries.
Details appear in [keys and commands](../reference/keyboard-and-slash-commands.md), [tool protocol](../reference/tool-protocol.md), and [HTTP API](../reference/http-api.md).
[Known limitations](../reference/known-limitations.md) and the [documentation index](../README.md) provide starting points for following up on unverified items.