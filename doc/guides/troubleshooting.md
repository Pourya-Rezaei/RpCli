# Troubleshooting with Minimal Risk

First record the application version, Node.js version, command run, and error text with secrets removed.
Do not include full tokens, configuration, or transcripts in an issue for diagnosis.

## The Application or Build Does Not Start

```sh
node --version
rc --version
```

Node.js 22+ is recommended for the current dependency set, although the declared minimum is 18.
A missing `rc` may be due to the global installation path; check your installation and PATH.
During development, missing `pnpm` or dependencies prevents the build from running; it is not evidence of broken code.
Setup commands appear in the [development guide](../development/setup-and-scripts.md).

## The Token Is Not Accepted

The application uses a web session token; do not substitute an official API key.
Saving through the form does not perform online validation.
The `DEEPSEEK_TOKEN` environment value can override the configuration file value.
Do not print the token in diagnostic output; check only the source of its configuration.
HTTP 401 or certain API messages are converted into token errors; 403 alone does not have that meaning.
The [configuration reference](../reference/configuration-and-storage.md) explains precedence.

## The Model, Search, or Response Stops

Check actual command names with `/help` and the current mode with TAB.
In `plan`, shell execution and file changes are blocked in the controlled execution path.
Search is not supported for a non-default model; check the `/search` setting.
Do not confuse the 10-tool-round limit with the transport continuation limit; `/continue` sends a fresh turn.
Ctrl+C during generation requests a stop, but not all stages, including PoW solving, are cancellable.
No overall request deadline is implemented; continued waiting does not necessarily indicate a tool loop.

## Data or Display Is Unexpected

Disabled logging does not mean that no local transcript exists.
`/clear` does not clear todos or remove the entire previous display.
For incomplete resumption, see [sessions and models](resume-and-model-switching.md).
Persian display depends on the terminal's bidi support; stored text retains logical order.
When reporting a problem, include the terminal type and a non-sensitive sample with the expected output.
`@path` does not automatically attach file content; ask the model to read the file.
For the server, a `/health` response does not guarantee working completions; see the [HTTP guide](http-server.md).

## Evidence and Validity

Token owner: [source/core/TokenConfig.ts](../../source/core/TokenConfig.ts), `tokenConfigPath`.
Transport owner: [source/core/apiClient.ts](../../source/core/apiClient.ts), `sendMessage` and `stopCurrentGeneration`.
Display owner: [source/core/bidiText.ts](../../source/core/bidiText.ts); adjacent tests describe transformation behavior.
Risks are listed in [known limitations](../reference/known-limitations.md), and reading paths are in the [index](../README.md).
Baseline version: **0.10.1**; reviewed: **2026-10-28**; static analysis, without runtime testing of DeepSeek.