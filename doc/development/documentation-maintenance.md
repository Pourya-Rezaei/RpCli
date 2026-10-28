# Maintaining Evidence-Based Documentation

This guide helps prevent tutorials from drifting away from the current version's behavior.
Every behavioral claim must identify its code owner, version, and validation method clearly.

## Content Ownership

Tutorials provide a short learning path with a verifiable outcome.
Guides solve a specific task; do not repeat the complete contract table.
References own options, types, defaults, and precise limitations.
Architecture explanations should describe rationale and flow, not replace execution instructions.
The reader's entry point is the [documentation index](../README.md).

## From Code Changes to Documentation Changes

For options and routing, start with [source/cli.tsx](../../source/cli.tsx), `cli`, and `firstArg`.
Reflect changes in the [CLI reference](../reference/cli.md) and affected tutorial.
For commands, the owner is `slashCommands` in [source/commands/index.ts](../../source/commands/index.ts).
Keep the complete list only in [keys and commands](../reference/keyboard-and-slash-commands.md).
For storage, inspect `tokenConfigPath` and `saveModelPreference` in [source/core/TokenConfig.ts](../../source/core/TokenConfig.ts).
Changes to them belong in [configuration and storage](../reference/configuration-and-storage.md).
For tools, read the registry together with the dispatcher and parser, not just the prompt text.
Descriptions are owned by `getToolDescriptions` in [source/tools/registry.ts](../../source/tools/registry.ts).
For HTTP, cross-check routes against `buildCompletionPrompt` and the session queue.
Input is owned by [source/server/request.ts](../../source/server/request.ts); the contract is in the [HTTP reference](../reference/http-api.md).

## Checklist for Every Review

1. Record the version and date from the actual state; a new date does not mean new testing.
2. Read the decision-making symbol, not just help, README, or the model's explanation.
3. Distinguish runtime-enforced behavior from prompt instructions.
4. Specify the working directory, side effects, and token requirements for shell examples.
5. Use loopback for the server and placeholders for secrets.
6. Check that link targets exist and code fences are closed.
7. Accompany test claims with actual commands and results; describe static analysis separately.
8. Keep unresolved discrepancies visible in [known limitations](../reference/known-limitations.md).

## Documentation Pitfalls in This Version

Do not call the web token an official API key or present `DEEPSEEK_API_KEY` as part of the application's contract.
Do not use `--plan` or mode-switching slash commands as examples of existing features.
Do not conflate optional logging with the persistent interactive transcript.
`-a` does not stage files when generating commit messages.
Do not present a server based on the last message and a shared session as fully OpenAI-compatible.
Distinguish the declared Node.js 18 minimum from the dependency-based recommendation of 22+.
The current count is 12 defined tests, not a successful execution result.
Older versions of the README and agent guide may disagree with the code; the code is the basis for behavioral claims.

## Evidence and Validity

Prompts are loaded through [source/prompts/index.ts](../../source/prompts/index.ts), `SystemPrompt`; see the [prompt reference](../reference/prompts.md).
For execution coverage, see [testing](testing.md); for building, see [setup](setup-and-scripts.md).
Baseline version: **0.10.1**; review: **2026-10-28**; static analysis, without live DeepSeek testing.