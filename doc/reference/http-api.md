# HTTP API Reference

Baseline version `0.10.1`, statically reviewed on `2026-10-28`. Responses and examples were derived from code, not from running the DeepSeek service. This interface is a stateful adapter, not a complete OpenAI implementation.

## Responsible Sources

[index.ts](../../source/server/index.ts): routes and `startServer`; [request.ts](../../source/server/request.ts): prompt construction and function parsing; [session.ts](../../source/server/session.ts): queue and sessions; [stream.ts](../../source/server/stream.ts): SSE; [types.ts](../../source/server/types.ts): TypeScript shapes; [http.ts](../../source/server/http.ts): CORS and tokens; [errors.ts](../../source/server/errors.ts): errors.

## Startup and Endpoints

The default bind address is `127.0.0.1:3000`. The port option takes precedence over `PORT`; only integers from 1 to 65535 are accepted, and invalid values fall back to 3000. The host option takes precedence over `HOST`. The JSON body size limit is configured as `10mb`.

| Method and Path | Behavior |
| --- | --- |
| `GET /health` | Status ok, object health, and timestamp; does not test DeepSeek connectivity |
| `GET /v1/models` | A list containing one model, `deepseek`, with created 0 and owned_by deepseek |
| `GET /v1/models/deepseek` | The same model object |
| `GET /v1/models/:model` for another name | 404 with model_not_found |
| `POST /v1/chat/completions` | Continue the active session or create a session as needed |
| `POST /v1/chat/completions/:sessionId` | Attempt to select the specified session |
| `OPTIONS *` | Respond with 204 after CORS middleware |

## Completion Input

| Field | Actual Behavior |
| --- | --- |
| `messages` | Nonempty array; only the last item is used to construct the prompt |
| Last `content` | String or text extracted from an array; trimmed text must not be empty |
| Last `role: tool` | Text is wrapped in a tool_result block with name and tool_call_id |
| `model` | Response label and thinking default; does not directly select Expert |
| `thinking_enabled` | When supplied, takes precedence over reasoning_effort and the model default |
| `reasoning_effort` | none/minimal disable thinking; low/medium/high/xhigh/max enable it; actual budget levels are not applied |
| `search_enabled` | Defaults to false; upstream search |
| `stream` | Only the exact value true enables SSE |
| `stream_options.include_usage` | When true, a final usage chunk with empty choices |
| `n` | Absent or 1; other values produce 400 |
| `tools` | Functions with valid names; absence uses the session cache if available |
| `tool_choice` | auto, none, required, or selection of a named function |
| `temperature`, `max_tokens` | Present in the type but not sent to the transport |

There is no complete JSON Schema validation for the body; a field's presence in a type does not guarantee runtime validation. Content arrays extract text only; image processing is not implemented. Previous messages, including system messages, are not replayed to construct the prompt.

`resolveModel` treats an absent name or `deepseek-chat` as not thinking; `deepseek-reasoner` and names matching reasoner or R1 enable thinking by default. Other names are also accepted as labels; the model list does not promise that all such models exist.

## JSON Response

```json
{
  "id": "chatcmpl-<generated>",
  "object": "chat.completion",
  "created": 0,
  "model": "deepseek-chat",
  "system_fingerprint": null,
  "choices": [{
    "index": 0,
    "message": {"role": "assistant", "content": "<answer>", "refusal": null},
    "logprobs": null,
    "finish_reason": "stop"
  }],
  "usage": {"prompt_tokens": 0, "completion_tokens": 0, "total_tokens": 0}
}
```

This example illustrates the response shape; created is actually a timestamp in seconds, and id is random. reasoning_content is added when thinking is enabled or reasoning exists. For a function call, message contains tool_calls; content can be null when there is no text, and finish_reason can be tool_calls. Usage zeros may be fallbacks for missing information rather than actual zero consumption.

The `X-RP-Session-Id` header returns the session ID if it is available before headers are sent. For continuation, a path containing sessionId is more explicit than relying on the global session; it still does not provide tenant isolation.

## SSE

Events are sent as `data: <JSON>` with object `chat.completion.chunk`; an assistant-role delta comes first, followed by content or reasoning_content. The final chunk contains finish_reason, and the stream closes with `data: [DONE]`. With include_usage, a chunk with empty choices precedes DONE.

When client tools are enabled, the stream is buffered until parsing finishes; this path is not true token-by-token streaming. An error after headers begin is sent as an error in the SSE body followed by DONE; the initial HTTP status can no longer be changed.

## Functions and State

Function names must match `[A-Za-z0-9_-]{1,64}`. Parameter definitions are provided to the model, but no schema validator runs. A call with an unknown name produces a server error. required without any call produces an error; choosing a named function rejects a call with a different name but does not necessarily reject the absence of all calls.

The server does not execute any of these functions through the local registry. The client is responsible for execution and sending results. Tool definitions are cached per session in an in-memory Map and are lost on restart. See [tool-protocol.md](tool-protocol.md) for format details.

All completions pass through one queue and use shared API state. Changing the token resets active state; selection and prompt/cache construction happen before queue execution. This architecture is not designed for an isolated multi-user service. Client disconnection stops output delivery; the upstream stop attempt happens in finally after awaiting the result, not necessarily at disconnection time.

## Compatibility and Errors

| Expected OpenAI Capability | Project Status |
| --- | --- |
| Completion envelope and SSE | Limited support |
| reasoning_content | Extension not compatible with every client |
| Replay all messages and the system role | Unsupported |
| Actual model selection through model | Approximate; mostly a label and thinking |
| Function calling | Text conversion and limited validation; client-side execution |
| Multiple choices, images, sampling controls | Unsupported or not forwarded |
| Independent session for every request/user | Not guaranteed |
| Separate local API authentication | Not implemented |

The error envelope contains `error.message`, `type`, `param`, and `code`. Invalid messages or JSON produce 400, a missing model/path produces 404, an invalid token produces 401, and general errors usually produce 500. A result interruption in a non-stream response can produce 504; this timeout does not imply a general request deadline. Token-error text uses OpenAI-compatible API key wording, but the project's actual credential is a web token.

Environment tokens take precedence over Bearer tokens; wildcard CORS and private-network allowance do not provide access protection. Before changing the bind address from loopback, read [Security](../explanation/security-and-privacy.md) and the [Server Guide](../guides/http-server.md).