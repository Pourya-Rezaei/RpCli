# Execution and Reporting Instructions

Date: `2026-10-28`. Scope: the Open WebUI integration described by [WEB-002](../web-development/architecture-decision.md).

## Before Acting

Read the [handoff index](README.md), [checklist](checklist.md), and [progress record](progress.md). Confirm the branch, tracked and untracked changes, versions, available commands, and existing processes without revealing secrets. Earlier environment observations may be stale. Do not reset the worktree or assume another session's edits are yours.

Compare instructions with actual code. Preserve current terminal, prompt, commit, resume, and serve behavior. Use the existing DeepSeek web-token/PoW path. Open WebUI runs independently; no custom frontend, alternative agent, or new state package is planned.

## The Task Loop

1. Select the first incomplete task whose prerequisites and authorization are satisfied. Only one task may be in progress.
2. Before acting, record the task ID, intended action, allowed scope, and cheapest meaningful verification in the progress snapshot.
3. Make the smallest scoped change or perform the bounded check. Do not bundle unrelated tasks.
4. Run focused validation immediately after a substantive edit. A failing check requires local diagnosis or a blocker report, not a completed checkbox.
5. Append a work-log entry with actions, changed files, exact checks, actual results, limitations, and next action. Update the snapshot and checklist evidence reference.
6. Mark the task complete only when its acceptance criteria are satisfied. Move to the next task only within the currently approved gate; otherwise stop for owner review.

Read-only inspection can be reported as one coherent task rather than logging every file read. Report every substantive edit, test run, service/configuration action, and failed attempt before beginning a different meaningful task. A shell command succeeding is not proof that the requested behavior works.

## Authorization Boundaries

Initial authorization covers read-only reconnaissance and updating these execution records. It does not cover dependency installation, download/setup of applications, starting persistent services, configuration of credentials, real DeepSeek requests, or product changes.

Before those actions, obtain approval specifying the Open WebUI release, native/container setup, reachable local addresses, service authentication boundary, allowed files, tests, and permitted network use. Do not fill unknown security or deployment choices with hidden defaults. Configuration-only experimentation may follow once explicitly approved; a demonstrated need for an adapter requires separate review before code changes.

Never request secrets in reports or model-visible forms. The owner supplies credentials directly through an appropriate local secret input. Logs, screenshots, fixtures, and committed documents must not contain tokens, private conversation content, or sensitive environment values.

## Stop Conditions

- No approved installation/version/network or authentication choice exists for the next action.
- Stock Open WebUI cannot safely support the limited chat experiment.
- A fix would change legacy behavior, introduce an adapter or dependency outside scope, or modify Open WebUI source.
- Baseline failures prevent distinguishing a regression; tests fail repeatedly or evidence contradicts the design.
- Session contamination, unexpected auxiliary requests, unauthorized file operations, or credential exposure is observed.
- A milestone's owner gate is reached, or the next milestone has not been authorized.

Stop the relevant activity safely, preserve evidence without secrets, mark the item blocked or awaiting review, and propose the smallest resolution. Do not bypass approvals or weaken tests. If a secret is exposed, do not reproduce it in the report; request appropriate revocation through the owner.

## Evidence Standards

Record commands with cwd, exit status, relevant sanitized output, and what the result demonstrates. For browser checks, record application/version, steps, observed behavior, and sanitized artifact location if available. Mock, real-network, static, and unexecuted checks must be labeled separately.

Routine tests use controlled mocks. File tests use temporary workspaces; commit tests use disposable repositories. Never run deletion or overwrite checks against user files. Real DeepSeek smoke tests require authorization and must preserve the existing transport rather than bypassing RpCli.

Do not report full compatibility from endpoint discovery, a screenshot, or one successful response. M1 is limited chat; M2 validates session correctness. Known unsupported operations must remain unsupported rather than appearing functional.

## Review and Session End

Self-verification is not owner approval. At a gate, summarize evidence and risks and wait. Do not mark future stages started based on a prior general request to proceed step by step.

At the end of a session update the snapshot, log the latest result, identify the exact next authorized action, and list running processes and how they are managed. Stop temporary services no longer needed; explicitly hand off intentionally retained services without exposing credentials. Record uncommitted files and pending approvals. Do not commit or push unless explicitly requested.