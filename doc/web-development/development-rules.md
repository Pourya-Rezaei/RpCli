# Integration Rules

Date: `2026-10-28`. Status: acceptance requirements for [WEB-002](architecture-decision.md), not implemented functionality.

## Scope and Reuse

1. Use Open WebUI as a separate maintained application. Do not scaffold a custom frontend or modify Open WebUI source by default.
2. Preserve the existing DeepSeek web-token, PoW, and transport path. Do not switch to a direct provider connection that bypasses RpCli.
3. Preserve bare `rc`, terminal, prompt flags, commit, resume, and the current serve contract. New behavior must be opt-in and additive unless explicitly approved.
4. Reuse the existing agent, parser, executor, file tools, and confirmation logic. Do not copy them into a plugin or implement a second tool loop.
5. Prefer configuration. Demonstrate and test a compatibility gap before introducing a small adapter. Backend changes need a specific reason and regression coverage.
6. Do not add Redux, Zustand, a new frontend, a replacement database, or a new model SDK without an approved unmet requirement.

## Initial Experiment

The first milestone permits controlled text chat and streaming only. Use a single conversation, one active request, and no coding tools. Disable auxiliary title, tag, suggestion, or other model-generation tasks for the experiment; verify actual requests rather than relying only on settings labels.

Do not claim multi-chat isolation, replay, edit, regeneration, file access, or full OpenAI compatibility. If stock Open WebUI cannot enforce the limited scope, report that limitation and stop before real-data testing. Do not silently patch either project to make a demonstration appear successful.

## Conversation and Run Ownership

The integration must distinguish Open WebUI chat IDs, RpCli remote session IDs, message IDs, and active run IDs. A serialized queue is not isolation. Selecting New Chat must not continue the previous remote conversation. Auxiliary requests must not mutate user-chat context.

If session mapping is needed, define ownership, lifetime, restart behavior, and allowed concurrent activity before implementing it. Keep Open WebUI history and DeepSeek history distinct. UI transcript edits do not prove that the upstream parent or context changed.

Cancellation must address the intended run; late chunks and callbacks must not update a newer run. Disconnect is not successful completion. Identify existing transport operations that cannot be interrupted rather than promising universal cancellation.

## Tool and Approval Contract

File operations are later milestones. Read first, then create, edit, and delete. Shell execution, elevation, broad workspace access, and automatic approval are outside this sequence.

For each mutating operation, the backend must bind approval to the actual operation, run, workspace, and intended path/content. Reject stale, repeated, cross-run, or altered approvals. Denial and disconnect must not default to approval. The browser may request an operation but cannot grant itself filesystem authority.

Preserve the current executor's batch failure/decline behavior and normal/plan policies. Do not activate yolo for the web integration without a new decision. Use existing preview helpers where suitable, and do not describe a simplified preview as a complete safety guarantee.

The current `write_file` tool overwrites existing paths. A create milestone must explicitly test and document overwrite behavior; do not label it create-only unless a separately approved restriction enforces that contract.

## Security and Data

- Keep services local by default. Open WebUI authentication does not automatically secure direct access to RpCli.
- Define network reachability for the selected native or container deployment. Container localhost is not the host's loopback; do not solve reachability by exposing an unauthenticated RpCli endpoint publicly.
- Keep the DeepSeek credential in RpCli's server environment/configuration. It is not the Open WebUI user password or an independent local-service access key.
- Do not commit secrets, conversation databases, volumes, transcripts, or personal connection configuration.
- Do not mount the entire host filesystem into Open WebUI to simulate reuse of RpCli tools.
- Community plugins can execute privileged backend code. Review dependencies, authentication, and permissions before installation.
- Existing file-path and shell risks remain; see the [security analysis](../explanation/security-and-privacy.md). Integration does not create a sandbox.

## Test and Change Discipline

Each milestone adds focused automated tests and a reproducible acceptance procedure. Mock the DeepSeek boundary for routine tests. Use temporary files and repositories for mutations; never test destructive behavior on user files. Run a separately authorized real-DeepSeek smoke test where relevant, and label it distinctly from mock coverage.

Preserve existing build, lint, and test gates. Record exact commands, results, environment versions, and tests not run. Do not install dependencies, change global configuration, register upstream remotes, commit, push, or broaden scope without authorization for that action.

Keep implementation diffs focused. Do not clean up unrelated code, change legacy API semantics, or hide failing tests. Every dependency or configuration change must state how it affects [updates](update-policy.md).

## Execution Handoff

An execution agent must first read this directory and AGENTS guidance, inspect the actual worktree, and execute only the approved milestone from the [delivery plan](delivery-plan.md). Existing edits must be preserved. When blocked by unsupported session or approval behavior, report evidence and the smallest proposed resolution; wait for approval before moving to another milestone.