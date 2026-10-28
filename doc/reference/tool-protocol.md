# Tool-Calling Protocol

Version `0.10.1`, statically reviewed on `2026-10-28`. This page describes message formats; it is not an instruction to execute operations.

## Two Independent Protocols

| Path | Model Format | Executor |
| --- | --- | --- |
| Local agent | `tool_calls / invoke / parameter` | Local registry through the executor |
| HTTP functions | `function_call / argument` | HTTP client; the server only translates |

These two formats are not interchangeable. Local-tool names and inputs are documented in [local-tools.md](local-tools.md).

## Local Format

Owner: `parseToolCalls` in [parse.ts](../../source/tools/parse.ts), with type definitions in [types.ts](../../source/tools/types.ts).

```xml
<tool_calls>
<invoke name="read_file">
<parameter name="path">README.md</parameter>
<parameter name="offset">1</parameter>
<parameter name="limit">20</parameter>
</invoke>
</tool_calls>
```

The parser is regex-based; it is not a general XML parser with full escaping, nesting, and schema support. Parameter values start as strings; `stringArgument`, `textArgument`, and `positiveIntegerArgument` in [arguments.ts](../../source/tools/arguments.ts) perform tool-specific interpretation and validation. Input whitespace and newlines must be checked against the relevant helper's rules; inputs are not all trimmed identically.

In this version, an invoke without a parameter is not accepted as a valid call. Tools such as `todo_list()` and `todo_clear()` are therefore incompatible with this markup path despite being defined without parameters in the registry. This is a parser limitation, not an absence of the tools from the registry.

## Normalization and Agent Loop

[toolCallMarkup.ts](../../source/core/toolCallMarkup.ts) converts upstream DSML to the local format and handles incomplete fragments for display. This normalization is not equivalent to parsing and authorized execution.

`getAIResponse` in [agent.ts](../../source/actions/agent.ts) checks answer for tools first, then thinking if answer contains no calls. Output hidden in the UI can still inform the agent's decisions on this path.

1. Receive the model response and normalize markup.
2. Extract calls or identify a parse error.
3. Run the executor with the mode and confirmation callback.
4. Send successes and errors to the model in `tool_result` blocks.
5. Continue until a final answer, a stop, or the limit of 10 tool rounds.

A parse error also consumes the tool-round budget and sends a format-correction message to the model. At the limit, the agent requests a summary without tools; this limit differs from the transport layer's incomplete-response continuation limit. An absent confirmation callback does not grant default permission for operations.

## HTTP Function Format

Owners: `withToolInstructions` and `parseClientToolCalls` in [request.ts](../../source/server/request.ts).

```xml
<function_call name="lookup_document">
<argument name="document_id">"example"</argument>
<argument name="limit">3</argument>
</function_call>
```

Argument values are first tried with `JSON.parse` and fall back to strings on failure. A JSON object is also accepted as the function body; the parser has a fallback path for triple-quoted strings. External output is a call with a generated ID and `function.arguments` as a JSON string.

The function name must occur in the request tool definitions or session cache. The supplied parameter JSON Schema is not enforced as a runtime validation contract. The client's result `tool_call_id` is not matched against a recorded call either. The named-tool selection limitation is recorded in [known-limitations.md](known-limitations.md).

In the client's next response, `buildCompletionPrompt` wraps the last message with role `tool` in `tool_result`. Sending several results in messages does not mean all are processed; only the last one forms the basis of the prompt.