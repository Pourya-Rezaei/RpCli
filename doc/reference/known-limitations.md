# Known Limitations and Discrepancies

Reviewed version `0.10.1`, dated `2026-10-28`. This list results from static analysis; it is neither an exploitation report nor a runtime test result. Code and root documentation were not modified as part of this work.

## How to Read This List

"Confirmed from code" means the relevant control flow or configuration is visible; a "static risk" requires a dedicated test. Priority indicates importance for decisions and use, not a formal security score. Each row has an ID for reference in issues or subsequent reviews.

## Execution, Safety, and Data

| ID | Priority | Finding | Evidence and Consequence |
| --- | --- | --- | --- |
| SEC-01 | High | Shell is not confined; confirmed from code | [registry.ts](../../source/tools/registry.ts): exec without sandbox or timeout; review confirmations carefully |
| SEC-02 | High | Final write-target symlink; static risk | [paths.ts](../../source/tools/paths.ts): safeTargetPath only resolves the parent's realpath; preview may block earlier in normal mode |
| SEC-03 | High | Search over Git-listed files; static risk | [registry.ts](../../source/tools/registry.ts): no independent safePath per file, and fs.stat/readFile can follow symlinks |
| SEC-04 | High | Server with a configured token lacks independent client authentication | [http.ts](../../source/server/http.ts): environment takes precedence over Bearer; a public port is unsuitable without external controls |
| SEC-05 | High | Shared session and queue without tenant isolation | [session.ts](../../source/server/session.ts): global state and in-memory cache; queuing does not mean isolation |
| SEC-06 | Medium | sudo detection is not shell-aware | [sudo.ts](../../source/tools/sudo.ts): regex and text rewriting can change quotation semantics |
| DATA-01 | Medium | Disabling logging does not prevent transcript storage | [SessionTranscript.ts](../../source/core/SessionTranscript.ts): independent location for resume |
| DATA-02 | Medium | Completion of a deletion request does not prove deletion | [DeleteSession.ts](../../core-lib/DeleteSession.ts): HTTP success is not asserted |
| PATH-01 | Low | Some names within the workspace are rejected too broadly | [paths.ts](../../source/tools/paths.ts): startsWith('..') extends beyond a parent segment |

Threat descriptions and the limits of these claims are in [Security and Privacy](../explanation/security-and-privacy.md). This page is only a risk index.

## Compatibility and Behavior

| ID | Finding | Evidence and Effect |
| --- | --- | --- |
| ENV-01 | Declared engine and dependency requirements conflict | [package.json](../../package.json) specifies Node >=18; Ink 7 in [pnpm-lock.yaml](../../pnpm-lock.yaml) requires Node >=22; the practical 22+ recommendation comes from dependencies, not testing every version |
| ENV-02 | Potential peer/type mismatch | React 19, @types/react 18, and differing peers in some Ink dependencies; possible incompatibility, not proof of build failure |
| TOOL-01 | Invocations without parameters are not parsed | [parse.ts](../../source/tools/parse.ts): incompatible with argument-free todo tools |
| TODO-01 | Todo tree is neither general-purpose nor persistent | [todo.ts](../../source/tools/todo.ts): one-level display/completion and nonrecursive cleanup of split children |
| API-01 | Only the last request message is processed | [request.ts](../../source/server/request.ts): full history and system-role replay are not performed |
| API-02 | model does not actually select Expert | [request.ts](../../source/server/request.ts) and [session.ts](../../source/server/session.ts): label and thinking default |
| API-03 | Named tool does not fully enforce the zero-call case | [server/index.ts](../../source/server/index.ts): some on an empty array is false; the required string has a separate check |
| API-04 | Function schema and tool-result ID are not enforced | [request.ts](../../source/server/request.ts): limited validation, not a sandbox for results |
| API-05 | Tool stream is buffered | [server/index.ts](../../source/server/index.ts): output after parsing, not token-by-token |
| API-06 | Disconnect does not immediately cancel upstream work | Same file: stop in finally after await; no general deadline either |
| MODEL-01 | Retry of a thinking-only response does not preserve the model | [apiClient.ts](../../source/core/apiClient.ts): final retry requests omit modelType |
| BUILD-01 | Build is not clean, and watch does not continually copy prompts | [copy-prompts.mjs](../../scripts/copy-prompts.mjs) and package scripts |
| UI-01 | Markdown rendering supports a subset | [MarkdownText.tsx](../../source/components/MarkdownText.tsx): neither browser output nor support for all tokens |
| UI-02 | Keyboard and bidi depend on the terminal | [bidiText.ts](../../source/core/bidiText.ts), [useComposerKeys.ts](../../source/components/chat/useComposerKeys.ts); inline code does not have the code-block exemption |

## Discrepancies with Root Documentation

| Existing Document | Claim or Gap | Correct Basis for This Documentation Set |
| --- | --- | --- |
| [README.md](../../README.md) | --plan, /plan, /normal, and /yolo | Not registered in the CLI/registry; TAB changes the mode |
| Same document | /clear clears the todo list | Chat reset and transcript deletion are not the same as todo_clear |
| Same document | PageUp/PageDown scroll by a page | Handler sends +/-1 |
| Same document | Redirection includes only the final response | quiet only controls thinking display; structured stdout is not guaranteed |
| Same document | General claims about path restrictions and confirmation | Shell, symlinks, and modes have important exceptions |
| [AGENTS.md](../../AGENTS.md) | No tests; use DEEPSEEK_API_KEY | 12 tests are defined; the application requires the web DEEPSEEK_TOKEN |
| Same document | Prompt destination and clean build | Output is dist/source/prompts; no clean step exists |
| Same document | Server routing in App and enforced Conventional Commits | Server is selected in the CLI; commit format is primarily a prompt instruction |

These discrepancies are recorded to prevent confusion; old text was not rewritten without permission. When correcting an item, update its ID, evidence, and related reference page together.

## Items Not Yet Verified at Runtime

Connection to a real DeepSeek account, stream behavior under web API changes, all terminals, cross-platform elevation, cancellation, symlinks, and concurrent clients were not exercised in this work. Existing tests are limited to bidi, markup, and some tools; the [Testing Reference](../development/testing.md) explains criteria for expanding coverage.

The existence of this list does not imply that other areas are defect-free. See [CLI](cli.md), [Tools](local-tools.md), and [HTTP API](http-api.md) for current values and contracts.