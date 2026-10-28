# Repository Coverage and Evidence Map

Version `0.10.1`; dated `2026-10-28`. This list maps the inventory of 85 Git-tracked files at the start of documentation to their responsibilities and explanatory pages. New `doc/` files are not included in this number. "Coverage" here means documentation coverage, not test coverage.

For each change, find the owning file in the table and review the related page. A file reference is not sufficient proof of successful execution; execution status is recorded in the [Verification Report](../development/verification.md).

## Configuration, Packaging, and Root Guides

| File | Responsibility | Explanatory Reference |
| --- | --- | --- |
| [.editorconfig](../../.editorconfig) | Whitespace, LF, and encoding | [Development Environment](../development/setup-and-scripts.md) |
| [.gitattributes](../../.gitattributes) | Git rules for file types/line endings | [Development Environment](../development/setup-and-scripts.md) |
| [.gitignore](../../.gitignore) | Items excluded from tracking | [Discovery and Security](../explanation/security-and-privacy.md) |
| [.prettierignore](../../.prettierignore) | Formatter exceptions, including Markdown | [Verification](../development/verification.md) |
| [AGENTS.md](../../AGENTS.md) | Repository agent guide and prompt context | [Discrepancies](known-limitations.md) |
| [README.md](../../README.md) | Current introduction and user commands | [Discrepancies](known-limitations.md) |
| [package.json](../../package.json) | Version, dependencies, bin, scripts, and tool configuration | [Development Environment](../development/setup-and-scripts.md) |
| [pnpm-lock.yaml](../../pnpm-lock.yaml) | Resolved versions and engine/peer dependencies | [Environment Limitations](known-limitations.md) |
| [pnpm-workspace.yaml](../../pnpm-workspace.yaml) | Workspace configuration and release-age policy | [Development Environment](../development/setup-and-scripts.md) |
| [tsconfig.json](../../tsconfig.json) | source/core-lib input and dist output | [Development Environment](../development/setup-and-scripts.md) |
| [copy-prompts.mjs](../../scripts/copy-prompts.mjs) | Copy prompt assets after compilation | [Prompts](prompts.md) |

## Entry Points and Orchestration

| File | Responsibility | Explanatory Reference |
| --- | --- | --- |
| [cli.tsx](../../source/cli.tsx) | Parsing and serve/resume/commit/prompt/interactive selection | [CLI](cli.md) |
| [app.tsx](../../source/app.tsx) | Token gate, appropriate view, and exit cleanup | [Execution Flows](../explanation/execution-flows.md) |
| [agent.ts](../../source/actions/agent.ts) | Tool and response loop | [Protocol](tool-protocol.md) |
| [commitMessage.ts](../../source/actions/commitMessage.ts) | Diff and commit-message generation | [Commit Guide](../guides/commit-messages.md) |
| [commands/index.ts](../../source/commands/index.ts) | Slash-command registry | [Commands](keyboard-and-slash-commands.md) |

## Interface Components

| File | Responsibility | Explanatory Reference |
| --- | --- | --- |
| [CommitView.tsx](../../source/components/CommitView.tsx) | Confirm and execute git commit | [Commit Guide](../guides/commit-messages.md) |
| [FzfFilePicker.tsx](../../source/components/FzfFilePicker.tsx) | Internal fuzzy file picker | [Terminal Interface](../explanation/terminal-and-persian-rendering.md) |
| [InteractiveChatView.tsx](../../source/components/InteractiveChatView.tsx) | Main chat and input control | [Execution Flows](../explanation/execution-flows.md) |
| [MarkdownText.tsx](../../source/components/MarkdownText.tsx) | Interpret a Markdown subset for Ink | [Persian Rendering](../explanation/terminal-and-persian-rendering.md) |
| [RpCliLogo.tsx](../../source/components/RpCliLogo.tsx) | Interface identity and startup information | [Architecture](../explanation/architecture.md) |
| [SinglePromptView.tsx](../../source/components/SinglePromptView.tsx) | Single-request lifecycle and cleanup | [Execution Flows](../explanation/execution-flows.md) |
| [SlashCommandPicker.tsx](../../source/components/SlashCommandPicker.tsx) | Select registered commands | [Commands](keyboard-and-slash-commands.md) |
| [Spinner.tsx](../../source/components/Spinner.tsx) | Waiting indicator | [Architecture](../explanation/architecture.md) |
| [TokenSetupView.tsx](../../source/components/TokenSetupView.tsx) | Web-token entry form | [Installation](../tutorials/installation.md) |
| [ToolConfirmation.tsx](../../source/components/ToolConfirmation.tsx) | Display and resolve tool confirmation | [Local Tools](local-tools.md) |
| [ChatStatusBar.tsx](../../source/components/chat/ChatStatusBar.tsx) | Mode, search, thinking, and model | [Terminal Interface](../explanation/terminal-and-persian-rendering.md) |
| [MessageRow.tsx](../../source/components/chat/MessageRow.tsx) | Display chat roles | [Terminal Interface](../explanation/terminal-and-persian-rendering.md) |
| [ModelConfirmation.tsx](../../source/components/chat/ModelConfirmation.tsx) | Confirm model switching | [Resume and Models](../guides/resume-and-model-switching.md) |
| [PlanConfirmation.tsx](../../source/components/chat/PlanConfirmation.tsx) | Decide whether to start a plan | [Execution Flows](../explanation/execution-flows.md) |
| [createLiveStream.ts](../../source/components/chat/createLiveStream.ts) | Buffer and schedule display updates | [Execution Flows](../explanation/execution-flows.md) |
| [chat/types.ts](../../source/components/chat/types.ts) | Message types and display state | [Interfaces](exported-symbols.md) |
| [useChatSession.ts](../../source/components/chat/useChatSession.ts) | Session lifecycle | [Execution Flows](../explanation/execution-flows.md) |
| [useComposerKeys.ts](../../source/components/chat/useComposerKeys.ts) | Text editing and TAB | [Keys](keyboard-and-slash-commands.md) |

## Shared Core

| File | Responsibility | Explanatory Reference |
| --- | --- | --- |
| [apiClient.ts](../../source/core/apiClient.ts) | API state, PoW, and sending | [Transport](../explanation/deepseek-transport.md) |
| [bidiText.ts](../../source/core/bidiText.ts) | Persian visual ordering with environment detection | [Persian Rendering](../explanation/terminal-and-persian-rendering.md) |
| [Gitignore.ts](../../source/core/Gitignore.ts) | Fallback ignore rules | [Security](../explanation/security-and-privacy.md) |
| [ListWorkspaceFiles.ts](../../source/core/ListWorkspaceFiles.ts) | Git listing or fallback traversal | [Tools](local-tools.md) |
| [LogChat.ts](../../source/core/LogChat.ts) | Optional diagnostic logging | [Storage](configuration-and-storage.md) |
| [OpenDeepSeek.ts](../../source/core/OpenDeepSeek.ts) | Platform-specific browser opening | [Installation](../tutorials/installation.md) |
| [SessionTranscript.ts](../../source/core/SessionTranscript.ts) | Local transcript and ID validation | [Storage](configuration-and-storage.md) |
| [textCursor.ts](../../source/core/textCursor.ts) | Graphemes, positions, and words | [Persian Rendering](../explanation/terminal-and-persian-rendering.md) |
| [TokenConfig.ts](../../source/core/TokenConfig.ts) | Token and preferences | [Configuration](configuration-and-storage.md) |
| [toolCallMarkup.ts](../../source/core/toolCallMarkup.ts) | Normalize DSML and incomplete markup | [Protocol](tool-protocol.md) |
| [bidi-js.d.ts](../../source/types/bidi-js.d.ts) | Local declaration for the bidi dependency | [Persian Rendering](../explanation/terminal-and-persian-rendering.md) |

## DeepSeek Web Connection

| File | Responsibility | Explanatory Reference |
| --- | --- | --- |
| [Chat.ts](../../core-lib/Chat.ts) | Completion, SSE, and continuation | [Transport](../explanation/deepseek-transport.md) |
| [ChatSessions.ts](../../core-lib/ChatSessions.ts) | Session listing | [Transport](../explanation/deepseek-transport.md) |
| [ContinueChat.ts](../../core-lib/ContinueChat.ts) | Stream continuation | [Transport](../explanation/deepseek-transport.md) |
| [CreatePowChallenge.ts](../../core-lib/CreatePowChallenge.ts) | Challenge | [Transport](../explanation/deepseek-transport.md) |
| [CreateSessions.ts](../../core-lib/CreateSessions.ts) | Session creation | [Transport](../explanation/deepseek-transport.md) |
| [DeleteSession.ts](../../core-lib/DeleteSession.ts) | Deletion request | [Transport](../explanation/deepseek-transport.md) |
| [InvalidTokenError.ts](../../core-lib/InvalidTokenError.ts) | Invalid-token detection | [Transport](../explanation/deepseek-transport.md) |
| [StopStream.ts](../../core-lib/StopStream.ts) | Stop a remote response | [Transport](../explanation/deepseek-transport.md) |
| [core-lib/index.ts](../../core-lib/index.ts) | Barrel export | [Interfaces](exported-symbols.md) |

## Server and HTTP Contract

| File | Responsibility | Explanatory Reference |
| --- | --- | --- |
| [errors.ts](../../source/server/errors.ts) | ApiError and error envelope | [HTTP](http-api.md) |
| [http.ts](../../source/server/http.ts) | Token and CORS | [HTTP](http-api.md) |
| [index.ts](../../source/server/index.ts) | Endpoints and listener lifecycle | [HTTP](http-api.md) |
| [request.ts](../../source/server/request.ts) | Prompt, model, and function parsing | [HTTP](http-api.md) |
| [session.ts](../../source/server/session.ts) | Queue, initialization, and cache | [HTTP](http-api.md) |
| [stream.ts](../../source/server/stream.ts) | SSE construction | [HTTP](http-api.md) |
| [types.ts](../../source/server/types.ts) | Request and response types | [HTTP](http-api.md) |

## Local Tools

| File | Responsibility | Explanatory Reference |
| --- | --- | --- |
| [arguments.ts](../../source/tools/arguments.ts) | Conversion and validation | [Protocol](tool-protocol.md) |
| [describe.ts](../../source/tools/describe.ts) | Prepare confirmation previews | [Tools](local-tools.md) |
| [diff.ts](../../source/tools/diff.ts) | Simple display diff | [Security](../explanation/security-and-privacy.md) |
| [execute.ts](../../source/tools/execute.ts) | Dispatch, mode, and batch | [Tools](local-tools.md) |
| [index.ts](../../source/tools/index.ts) | Consumer-facing exports | [Interfaces](exported-symbols.md) |
| [parse.ts](../../source/tools/parse.ts) | Local markup | [Protocol](tool-protocol.md) |
| [paths.ts](../../source/tools/paths.ts) | cwd and containment checks | [Security](../explanation/security-and-privacy.md) |
| [registry.ts](../../source/tools/registry.ts) | Tool implementations and descriptions | [Tools](local-tools.md) |
| [sudo.ts](../../source/tools/sudo.ts) | Elevation detection and command rewriting | [Security](../explanation/security-and-privacy.md) |
| [todo.ts](../../source/tools/todo.ts) | In-memory tasks | [Tools](local-tools.md) |
| [types.ts](../../source/tools/types.ts) | Tool and confirmation contract | [Interfaces](exported-symbols.md) |

## Prompt Assets and Tests

| File | Responsibility | Explanatory Reference |
| --- | --- | --- |
| [continue.md](../../source/prompts/continue.md) | Continue work | [Prompts](prompts.md) |
| [executePlan.md](../../source/prompts/executePlan.md) | Execute an approved plan | [Prompts](prompts.md) |
| [gitMessage.md](../../source/prompts/gitMessage.md) | Commit-message instruction | [Prompts](prompts.md) |
| [prompts/index.ts](../../source/prompts/index.ts) | Builders and AGENTS | [Prompts](prompts.md) |
| [init.md](../../source/prompts/init.md) | Generate AGENTS | [Prompts](prompts.md) |
| [plan.md](../../source/prompts/plan.md) | Planning | [Prompts](prompts.md) |
| [server.md](../../source/prompts/server.md) | HTTP adapter context | [Prompts](prompts.md) |
| [system.md](../../source/prompts/system.md) | Agent context | [Prompts](prompts.md) |
| [tools.md](../../source/prompts/tools.md) | Tool format and instructions | [Prompts](prompts.md) |
| [bidiText.test.ts](../../source/core/bidiText.test.ts) | 5 bidi cases | [Testing](../development/testing.md) |
| [toolCallMarkup.test.ts](../../source/core/toolCallMarkup.test.ts) | 3 markup cases | [Testing](../development/testing.md) |
| [tools.test.ts](../../test/tools.test.ts) | 4 tool and markup cases | [Testing](../development/testing.md) |
| [lines.txt](../../test/fixtures/lines.txt) | Range-reading fixture | [Testing](../development/testing.md) |

## Verification Method

Check the inventory with `git ls-files` and compare it against this page's link targets. A new source file without a row indicates a need for review. Linking a helper file to its owning page does not necessarily mean it needs a separate document; documentation is separated by responsibility, not by file count.