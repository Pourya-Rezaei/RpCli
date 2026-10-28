# Development Stages and Acceptance Criteria

Date: `2026-10-28`; status: a stage roadmap, not an execution plan with estimates or authorization to begin coding. None of the product stages below were implemented in this work.

## Stage Zero: Resolve Open Decisions

Determine the web entry command and preservation of the `rc` default, frontend packaging, session model, and local access authentication method. The [architecture decision](architecture-decision.md) and [development rules](development-rules.md) form the review baseline. The output is a short approved design and executable acceptance tests, not scaffolding before decisions.

## Stage One: Web Shell and Chat

Create a separate frontend with assistant-ui and a minimal bridge. The goals are text submission, streaming responses, errors, and Markdown/code and Persian text rendering. A limited prototype may consume the current completion endpoint, but must make clear that agent tools are not enabled through that path.

Acceptance: install the artifact and run the launcher without a development server; open the browser or print a fallback URL; keep the token out of the browser; support one conversation and active run; correctly parse streams split across chunk boundaries; preserve old paths. UI controls without backend support must not be enabled.

## Stage Two: Lifecycle and Sessions

Define and connect run IDs, thread-to-remote-session mapping, history contracts, and cancellation. Do not equate browser display data with the remote parent. Do not conflate stopped, error, and complete statuses; late requests must not transfer to the next run.

Acceptance: refresh and authorized continuation, new conversations without previous-session leakage, upstream stopping within supported limits, tab disconnection during generation, process restart, and recovery according to a defined policy. Explicitly report any transport segment without cancellation support.

## Stage Three: Agent and Tool Approval

The bridge should run the existing agent with a web approval callback. Tool events and previews should use the corresponding existing logic; a browser caller may not construct arbitrary preapproved commands. Rejection, timeout, and process exit must have defined outcomes.

Acceptance: no write/edit/delete/command operation requiring approval runs before valid authorization; batch rejection remains consistent with the executor contract; reject stale approvals and approvals belonging to another run. Normal and plan modes must match the [current executor](../../source/tools/execute.ts). Web yolo remains out of scope without a separate decision.

## Stage Four: Experience Completion and Release

Expose model, thinking, search, todo, and path selection in the UI only after actual integration. File selection is not the same as upload or automatic content submission. Message editing, regeneration, branching, concurrent runs, and public deployment are later projects with separate backend prerequisites.

Acceptance: mobile/desktop, RTL/LTR, focus and keyboard, loading/error/empty states, stopping, and approval; include assets in the final package and do not publish the development template. Check occupied ports, unavailable browsers, missing tokens, and clean shutdown.

## Regression Matrix

| Path | Criterion |
| --- | --- |
| Running without arguments | Same current behavior unless an explicit decision changes the default |
| terminal | Works independently of the browser and frontend |
| prompt and flags | Existing output and settings match the CLI reference |
| commit | Diff and approval without unintended commits; test in a temporary repository |
| resume | Session and transcript continuation follow the previous contract |
| serve | No unintended changes to endpoints, envelopes, or client tools |
| DeepSeek | Same credentials/PoW/transport; distinguish real network tests from mocks |

## Testing and Reporting Policy

Unit-test the adapter and bridge without a real account; test the browser with a fake backend for streaming, approval, and cancellation. Test tools only in temporary directories with harmless commands. Tests requiring a real token must not run unintentionally in public CI or expose secrets in logs.

Preserve existing [development environment](../development/setup-and-scripts.md) and [testing](../development/testing.md) gates, and add frontend gates. Changing the previous test script must not remove existing steps. Each delivery must report actual commands and results, unexecuted checks, and changed files; structural document tests do not prove product functionality.

## Definition of Done for Each Stage

Respect the change boundaries, check that stage's criteria, do not hide unfinished capabilities, update documentation for implemented behavior, and record limitations. Only then change the stage to "implemented". Commit, push, and PR actions require an authorized request.

## Status of This Document Set

Only decision and rules documents have been prepared. No dependencies were installed, and no new CLI command, upstream remote, update bot, or UI was created. Current checks of these documents are limited to link destination existence, balanced code fences, and accessibility from the index; a review report for an earlier snapshot is not a web feature test result.