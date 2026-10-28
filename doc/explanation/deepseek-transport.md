# DeepSeek Transport, Streaming, and Response Continuation

Review date: 2026-10-28 | Version: 0.10.1
The evidence on this page comes from reading the source; live connectivity, token validity, and current service compatibility have not been tested.

## Service Used

`chat` in [Chat.ts](../../core-lib/Chat.ts) sends requests to `https://chat.deepseek.com/api/v0/chat/completion`.
This endpoint is part of the web session flow; it should not be described as the official API service that uses an API key.
The credential is sent in a header using the Bearer scheme, and the expected credential type is a web session token.
`TokenSetupView` in [TokenSetupView.tsx](../../source/components/TokenSetupView.tsx) provides the flow for obtaining this token.
Sending the token at this layer is not equivalent to comprehensive validation of all upstream responses.
Version and platform headers are hard-coded; the documentation does not guarantee the stability of the service's private contract.

## Two Coordination Layers

`sendMessage` in [apiClient.ts](../../source/core/apiClient.ts) decides which session and parent to attach a message to.
`chat` in [Chat.ts](../../core-lib/Chat.ts) controls the completion request and response-body reading.
The former also handles PoW and tool-markup normalization; the latter defines the `ChatResult` and `ChatStreamChunk` types.
This separation explains why network errors, tool-parsing errors, and tool-execution errors are not interchangeable.

## Parent Discovery and Session Creation

When `parentMessageId` is unknown, `sendMessage` obtains the session list from `chatSessions`.
This call is owned by [ChatSessions.ts](../../core-lib/ChatSessions.ts), and the session descriptor type is `ChatSession`.
If the current ID is found, `current_message_id` is selected as the parent.
Otherwise, `createSessions` in [CreateSessions.ts](../../core-lib/CreateSessions.ts) creates a fresh session.
The input ID therefore does not definitively lock execution to an existing session.
`setCurrentSessionId` and `resetChatSession` change only local module state, not remote history.
After the response, the new message ID is retained for the next send.

## Proof-of-Work Challenge

`createPowChallenge` in [CreatePowChallenge.ts](../../core-lib/CreatePowChallenge.ts) retrieves the required challenge.
In `sendMessage`, a `PowSolver` instance is initialized, and `solve` builds the response payload.
The payload is encoded with `JSON.stringify` and `btoa` and placed in the `x-ds-pow-response` header.
This local solving step is a request prerequisite, not model text generation.
Synchronous solving is not stopped by the completion's AbortSignal; its speed and cost were not measured in this review.
The presence of PoW does not imply local request rate limiting or guaranteed abuse prevention.

```text
session lookup/create -> challenge -> solver -> completion SSE
                                              -> continuation when INCOMPLETE
```

## Reading SSE

`chat` first checks the response content type and expects `text/event-stream`.
A non-streaming response is read as JSON; a business error can supply `bizCode` and error text in the result.
A missing body also produces a separate unsuccessful result.
In the streaming path, `TextDecoder` and a line buffer assemble body chunks for event processing.
The implementation recognizes both flat content and upstream fragments and patches.
`ChatStreamChunk` separates the `thinking` and `response` channels, with `messageId`.
Along with text, `ChatResult` collects status, model information, and any received usage data.
The presence of a usage field in the type does not guarantee its presence in every response.
This code does not perform general, comprehensive structural validation of upstream contracts.

## Normalization and Display

`createToolCallMarkupNormalizer` and `normalizeToolCallMarkup` in [toolCallMarkup.ts](../../source/core/toolCallMarkup.ts) bridge DSML and the local tool format.
`sendMessage` keeps separate normalizers for response and thinking and flushes the remaining buffer at the end.
This normalization is distinct from parsing and argument validation in the agent loop.
Terminal updates are later batched in `createLiveStream`; a network chunk boundary is not necessarily an interface-message boundary.
The protocol text is described in the [tool protocol reference](../reference/tool-protocol.md).

## Three Independent Limits

| Limit | Owner | Exact Meaning |
| --- | --- | --- |
| 10 continuations | `MAX_CONTINUATIONS` in [Chat.ts](../../core-lib/Chat.ts) | Continuing a response with `INCOMPLETE` status within one chat call. |
| 2 additional attempts | `MAX_EMPTY_RESPONSE_RETRIES` in [apiClient.ts](../../source/core/apiClient.ts) | Requesting a final response when only nonempty thinking has arrived. |
| 10 tool rounds | `getAIResponse` in [agent.ts](../../source/actions/agent.ts) | The model, tool, and result cycle; separate from transport continuation. |

`continueChat` in [ContinueChat.ts](../../core-lib/ContinueChat.ts) sends the request to continue a message.
Continuing an incomplete response is distinct from a user asking again to continue a task or using `/continue`.
In the retry specifically for a thinking-only response, `modelType` is not passed to `sendMessage` again.
In this branch, preserving the Expert selection is therefore not guaranteed, and the default may apply.
None of these limits is an overall deadline for the entire task.

## Errors and Cancellation

`assertValidTokenResponse` in [InvalidTokenError.ts](../../core-lib/InvalidTokenError.ts) checks 401 responses and qualifying API error messages.
A 403 response alone is not sufficient evidence of an invalid token for this detection.
`isInvalidTokenError` lets the interface react differently to credential errors.
`sendMessage` converts an `ok: false` result into a throwable error; this differs from an intentional stop.
In the AbortError path, `chat` returns a stopped result with `stopped`.
`stopCurrentGeneration` first aborts the controller and calls `stopStream` if a message ID is available.
The stop request is owned by [StopStream.ts](../../core-lib/StopStream.ts).
Completion and continuation calls have cancellation signals; session discovery and challenge retrieval do not have the same coverage.
No general request timeout is defined at this layer.

## Deletion and Model Restrictions

`deleteSession` in [DeleteSession.ts](../../core-lib/DeleteSession.ts) returns the fetch response and does not necessarily assert HTTP success for deletion.
`stopStream` likewise returns the fetch response without guaranteeing that the stop was accepted; a resolved Promise is not proof of a remote outcome.
In `chat`, search for a non-default model is explicitly rejected, as is thinking combined with vision.
The underlying type recognizes vision, but that does not imply an image workflow exists in the CLI or HTTP interface.
`ref_file_ids` is empty in the completion request; mentioning a file path in text is not uploading that file to DeepSeek.

## Relationship to Local HTTP

[stream.ts](../../source/server/stream.ts) produces SSE for the HTTP client; this stream is not the same connection as the incoming DeepSeek SSE.
Names such as `deepseek-reasoner` in [request.ts](../../source/server/request.ts) can change the thinking default, not necessarily the upstream Expert model.
In client-tool mode, output is buffered for final parsing rather than delivered through the usual incremental stream.
For details, see the [HTTP reference](../reference/http-api.md), [symbols](../reference/exported-symbols.md), and [limitations](../reference/known-limitations.md).
For the transport's place in the application, see [architecture](architecture.md), [execution flows](execution-flows.md), and the [index](../README.md).