# Using the HTTP Server Locally

This guide covers connecting a local client to the chat adapter, not public deployment.
Prerequisites are an installed `rc` and a securely configured DeepSeek web token in the environment or configuration file.

## Starting on Loopback

```sh
rc serve --host 127.0.0.1 --port 3000
```

This mode does not open the interactive token-entry form.
To check from another terminal:

```sh
curl http://127.0.0.1:3000/health
curl http://127.0.0.1:3000/v1/models
```

Do not treat the health response as proof of token validity or successful DeepSeek connectivity.
Model discovery advertises only the `deepseek` model.

## Sending a Simple Request

```sh
curl http://127.0.0.1:3000/v1/chat/completions \
  -H 'Content-Type: application/json' \
  -d '{"model":"deepseek","messages":[{"role":"user","content":"Just say hello"}],"stream":false}'
```

This example uses the server's configured token and contains no real secret in the command.
If no server token is configured, Bearer can supply the web token.
Selection precedence is `DEEPSEEK_TOKEN`, then `RC_TOKEN`, then Bearer.
The configured token is not independent client authentication; the requester is not required to match it.

## Contract and Limitations

Only the last message's text content is used as conversation input.
Sending a complete OpenAI history array does not independently replay that history.
All completions share one queue and active session; this is not a multitenant server.
The reasoner model name affects the thinking default, not Expert selection in the upstream service.
Parameters such as `temperature` and `max_tokens` are not sent upstream.
Tools declared by the client must be executed by that client, not by the server's local file tools.
Tool requests can buffer stream output until parsing finishes.
See the [HTTP reference](../reference/http-api.md) for the SSE contract and compatibility matrix.

The server has no TLS, rate limiting, or session-owner isolation, and its CORS policy uses a wildcard.
Do not expose it on a public network without external protection.

## Evidence and Validity

Route owner: [source/server/index.ts](../../source/server/index.ts), `startServer`.
Input owner: [source/server/request.ts](../../source/server/request.ts), `buildCompletionPrompt`.
Shared-state owner: [source/server/session.ts](../../source/server/session.ts), `enqueueCompletion` and `runCompletion`.
Token owner: [source/server/http.ts](../../source/server/http.ts), `getToken`; return to the [index](../README.md).
Baseline version: **0.10.1**; reviewed: **2026-10-28**; static analysis, without running the server or testing DeepSeek.