# Ordered Checklist and Approval Gates

Initial status: all implementation steps are unstarted. An unchecked item is not implemented. Use the [reporting rules](instructions.md) and reference work-log entry IDs from [progress](progress.md).

## Status Rules

Checkboxes mean evidence-backed task completion only. Use the gate table for owner acceptance. Do not check items in advance or mark unavailable tests as passed. Blocked work stays unchecked with a reason in the progress snapshot. Add narrowly scoped subtasks only within an approved milestone; scope expansion needs review.

| Gate | Permitted scope | Authorization | Owner acceptance | Evidence |
| --- | --- | --- | --- | --- |
| A: reconnaissance | Read-only inspection and execution-record updates | Authorized by this handoff | Pending | Not started |
| B: M0 environment preparation | Only the specifically approved setup actions | Not authorized | Pending | None |
| C: M1 controlled chat | Approved configuration/testing; no tools or unapproved adapter | Not authorized | Pending | None |
| D: M2 conversation correctness | Approved session investigation and targeted solution | Not authorized | Pending | None |
| E: M3 read files | Approved existing-agent tool integration | Not authorized | Pending | None |
| F: M4 create files | Approved creation/overwrite policy and confirmation bridge | Not authorized | Pending | None |
| G: M5 edit files | Approved exact-match and preview consistency work | Not authorized | Pending | None |
| H: M6 delete files | Approved bounded deletion integration | Not authorized | Pending | None |

Approval entries must cite an actual owner instruction, date, and limits. An executing agent may record that evidence, not invent it. A gate is not authorized just because its predecessor passed.

## A: Read-Only Reconnaissance

- [ ] A-01 Read all required documents and record contradictions or missing decisions.
- [ ] A-02 Inspect branch/worktree, installed runtimes, package managers, dependencies, and existing services; preserve user edits.
- [ ] A-03 Recheck the current completion/session code and applicable Open WebUI release documentation without installing anything.
- [ ] A-04 Propose exact release, installation method, local topology, authentication, allowed actions, and test locations; record owner decisions needed.
- [ ] A-05 Report evidence and request gate B/C authorization; do not install or start services while waiting.

Acceptance: a reproducible environment assessment and explicit proposed setup, with no secret disclosure or product mutation. Owner review is required before proceeding.

## B: M0 Environment and Baseline

- [ ] B-01 Record owner-approved release, setup method, network use, authentication boundary, paths, and permitted installations.
- [ ] B-02 Prepare only the approved local environment and isolated data locations; record versions and configuration without secrets.
- [ ] B-03 Run approved baseline build/lint/tests and record actual failures or unavailable checks separately.
- [ ] B-04 Verify local reachability and credential handling; request review of the completed baseline.

Acceptance: the [M0 criteria](../web-development/delivery-plan.md) are met. A known failure is not a pass; any exception needs explicit owner acceptance before subsequent work.

## C: M1 Controlled Chat

- [ ] C-01 Start only approved services and record version, bind, port, workspace, and process ownership.
- [ ] C-02 Configure the OpenAI-compatible connection to RpCli, not the direct DeepSeek provider.
- [ ] C-03 Disable auxiliary model tasks and verify requests; limit the experiment to one conversation and no tools.
- [ ] C-04 Add/run approved automated tests for discovery, requests, normal/fragmented SSE, non-stream output, errors, and disconnect.
- [ ] C-05 Test real Open WebUI browser behavior against a controlled mock backend and report results.
- [ ] C-06 Perform a separately authorized real-DeepSeek smoke test through RpCli; otherwise record this as unexecuted and keep acceptance pending.
- [ ] C-07 Verify legacy paths, summarize evidence/limitations, and request owner acceptance before M2.

Acceptance: visible incremental chat and correct failure handling, no exposed secrets or unsupported functionality claims. An adapter requirement triggers review before implementation.

## D: M2 Conversation Correctness

- [ ] D-01 Inspect identity propagation; propose and obtain approval for any required session mapping.
- [ ] D-02 Test successive turns, New Chat, return/reload/restart, and two distinct context markers.
- [ ] D-03 Test auxiliary requests, stop/disconnect, and late callbacks; document persistence and cancellation limits.
- [ ] D-04 Record no cross-chat contamination and request owner acceptance before tools.

## E: M3 Read Files

- [ ] E-01 Obtain approval for the existing-agent tool bridge and visibility contract.
- [ ] E-02 Test temporary-workspace reads, ranges, size limits, missing files, traversal, and symlinks.
- [ ] E-03 Verify result/error ownership and unchanged file contents; request owner acceptance.

## F: M4 Create Files

- [ ] F-01 Agree creation versus overwrite semantics and backend-bound approval behavior.
- [ ] F-02 Test bytes, collisions, missing parents, invalid paths, symlinks, denial, replay, altered payload, and disconnect.
- [ ] F-03 Verify no write before valid approval and no mutation after denial; request owner acceptance.

## G: M5 Edit Files

- [ ] G-01 Test unique/no/duplicate match, empty replacement, rejected paths, and denied approval.
- [ ] G-02 Test source changes after preview and concurrent external modifications.
- [ ] G-03 Verify exact intended edits and preservation on failure; request owner acceptance.

## H: M6 Delete Files

- [ ] H-01 Test temporary regular/missing files, directory rejection, invalid paths, and symlink behavior.
- [ ] H-02 Test approval, denial, replay, and disconnect; verify actual filesystem state.
- [ ] H-03 Run approved regression gates and submit final owner review with outstanding limitations.

Detailed acceptance for D-H is in the [delivery plan](../web-development/delivery-plan.md). None authorizes shell/elevation, public hosting, a custom frontend, automatic approval, or commit/push.