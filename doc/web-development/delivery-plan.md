# Staged Open WebUI Delivery Plan

Date: `2026-10-28`. Status: approved sequencing; individual milestones require execution authorization. No integration milestone is reported as implemented by these documents.

## Global Contract

Use the existing DeepSeek connection and agent, keep the current entry points, and do not build a frontend. Prefer configuration; justify a narrow adapter only when a test exposes a gap. Every milestone needs tests and explicit evidence before the next begins. Follow [WEB-002](architecture-decision.md), the [rules](development-rules.md), and the [update policy](update-policy.md).

## M0: Environment and Baseline

Inspect worktree state and installed tools; preserve all existing changes. Select an Open WebUI release and installation method, verify local-service reachability, and define safe credential handling. Inventory existing test commands and capture baseline failures without unrelated fixes.

Acceptance: versioned environment record, a local-only topology, a documented authentication boundary, authorized installation steps, and exact baseline command results. Missing prerequisites must be reported. No public bind, real token in test fixtures, or silent global installation is allowed.

## M1: Chat Connection Only

Start the current RpCli service and configure Open WebUI's OpenAI-compatible connection to it. Use the advertised model and one controlled conversation. Disable auxiliary title/tag/suggestion model generation and verify that it is actually absent from observed requests. Keep tools, attachments, edit, regeneration, and multi-chat claims out of scope.

Tests: discovery, text requests, normal and fragmented SSE, non-stream completion, malformed/error responses, credential failure, and client disconnect. Mock upstream responses in automated tests; do not make the routine suite depend on a real account. Browser acceptance must use Open WebUI, not an unrelated test chat page.

Acceptance: a submitted message and incremental reply are visible in the real interface, errors do not become false successes, secrets stay server-side, and a separately authorized DeepSeek smoke test verifies the unchanged transport path. Demonstrate that the original CLI and serve behavior remain intact. A screenshot or open page alone is insufficient.

Stop if stock configuration cannot support this limited experiment safely. Report the exact mismatch and proposed minimal adapter rather than implementing it without review. This milestone does not claim complete conversation or coding-agent equivalence.

## M2: Conversation Correctness

Test successive turns, New Chat, returning to an existing conversation, restart/reload behavior, and stopping a reply. Inspect whether Open WebUI propagates any usable identity and whether RpCli's session header is retained. Define mapping and persistence before implementing necessary isolation changes through a supported extension or additive adapter.

Tests: two chats with distinct context markers, repeated turns, auxiliary requests, late chunks after cancellation, stale IDs, connection loss, and process restart. Initially allow only one active generation; multiple chats do not imply simultaneous generation support.

Acceptance: no cross-chat context contamination, known local/remote history ownership, stable selection of the intended session, and accurate cancellation limitations. Editing, regeneration, branching, and concurrency remain disabled or explicitly unsupported until separately designed. Do not proceed to tools with unresolved session correctness.

## M3: Read Files

Connect the existing read tool through the existing agent/executor. Decide the supported extension/event bridge for tool visibility before implementation. Do not substitute Open WebUI's own filesystem service for RpCli behavior.

Tests in a temporary workspace: allowed file, missing file, line ranges, size cap, traversal/out-of-root path, outward symlink, and tool failure reaching the correct conversation. Never read real secrets to test rejection.

Acceptance: a valid file result reaches the same agent conversation, existing read restrictions hold, errors are visible, and no file content changes. Recheck relevant known path limitations before claiming a safe boundary.

## M4: Create Files

Reuse the existing write tool with a verified approval bridge. Explicitly define collision handling: current write_file can overwrite, so do not claim create-only semantics without approved enforcement.

Tests: correct bytes, missing parent, existing target, denied/out-of-root destination, final-component symlink, approval/denial, modified payload after preview, stale/repeated/cross-run approval, and disconnect while waiting. Use temporary files only.

Acceptance: intended file content is written only following valid approval; denial and failed approval leave files unchanged. Any known destination-symlink gap blocks a safety claim and must be reported for an approved targeted fix.

## M5: Edit Files

Reuse exact-match edit behavior and preview/confirmation logic. Tests: unique replacement, no match, duplicate match, empty replacement, rejected path, denied approval, changed source after preview, and concurrent external modification.

Acceptance: only the intended replacement occurs; failures and denial preserve contents. If preview and execution cannot be bound safely when source changes, stop and design the smallest guard rather than assuming the preview is authoritative.

## M6: Delete Files

Reuse the existing delete tool with explicit target approval. Tests: regular file, missing file, directory rejection, invalid path, symlink behavior, denial, replay, and disconnect. Check filesystem state rather than trusting a success message.

Acceptance: only the authorized file is removed, rejected operations preserve data, and existing execution policy remains intact. No recursive deletion or shell fallback is introduced.

## Regression and Reporting Gates

| Area | Required evidence |
| --- | --- |
| Bare rc and terminal | Existing interactive path remains available without Open WebUI |
| Prompt and flags | Existing option handling and output behavior preserved |
| Commit | Diff and confirmation tested in a disposable repository; no user commit created |
| Resume | Existing session/transcript behavior preserved |
| Serve | Existing routes, response envelopes, and client-tool semantics unchanged |
| DeepSeek | Existing token/PoW/transport retained; mock and real-network evidence distinguished |
| Open WebUI | Actual pinned application used for acceptance, with request behavior observed |

Each report lists changed files, test commands, actual results, unexecuted checks, version combination, and remaining limitations. Tests reduce risk; they do not guarantee zero defects. Existing 12 declared cases are not adequate coverage of this integration by themselves; see [testing](../development/testing.md).

Shell/elevation, upload/RAG, voice, images, public hosting, multi-user access, automatic approval, launcher changes, and a replacement state-management package are outside this plan. Commit and push require authorization.