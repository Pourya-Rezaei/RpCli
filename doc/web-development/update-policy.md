# Update Intake and Evaluation Policy

Date: `2026-10-28`; status: target policy, not configured automation.

## Two Different Paths

| Source | Intake method | What is not automatically guaranteed |
| --- | --- | --- |
| Original RpCli project | An upstream remote and integration of changes into the fork | Conflict-free merges and preservation of all custom behavior |
| assistant-ui | New dependency and lockfile versions | Compatibility of local adapters and components with the new API |

"Keep the update path open" means new versions can be obtained and reviewed, not unauthorized installation at runtime or automatic merging without tests. Upstream fixes reach consumers only after intake, integration, and publication of a new fork release.

## Receiving RpCli Changes

`origin` is the fork repository, and upstream must be the verified original repository. No remote registration or fetch was performed in this work. Verify the real URL and branch before configuration; do not assume main or master.

Maintenance workflow: start with a clean working tree, fetch upstream, review changes, integrate on a separate review branch, resolve conflicts and run gates, then merge into the fork's development branch. Do not rebase or force-push a shared branch without coordination. Receiving upstream changes must not cause a push to the original repository; contributions use a separate PR.

To reduce conflicts, do not move original files or copy core logic into a second fork, and keep the launcher and adapter small. Record necessary core patches with clear rationale and tests so the next version can establish whether they are still needed or upstream has resolved them.

## Receiving assistant-ui Versions

Use the official package as a dependency; do not vendor or fork the library without a new, justified decision. Record the resolved version in the lockfile; reproducible releases matter more than silently obtaining latest at runtime. Pinning a version does not block updates; updates should be deliberate.

Components copied into the project by a generator are local code and are not updated by package updates. Prefer official primitives and small wrappers. For copied files, record the source, original version, license, and custom changes, and schedule independent reviews. Generator updates must not overwrite customizations without review.

Check related assistant-ui packages together with React peer dependencies and runtime dependencies; incompatible single-package upgrades are not allowed. Minor or patch changes, especially before version 1, do not guarantee unchanged contracts; always review release notes and migration guides.

## Proposed Automation

Choose either Dependabot or Renovate later; both are not needed simultaneously. The tool should create version PRs, not merge without review. The proposed initial cadence is weekly and requires CI configuration approval; address important security issues outside this cycle.

Keep dependency PRs and upstream synchronization PRs separate where possible so failures can be traced. Record lockfile changes, API changes, test results, and rollback risks in the PR description. No bot, workflow, or secret was added at this stage.

## Gates for Every Update

1. Build/typecheck the core and frontend, and inspect release package contents.
2. Test existing terminal, prompt, commit, resume, and serve paths.
3. Test the adapter with fragmented streams, errors, and cancellation; use a fake backend without a real token.
4. Reject operations without approval, repeated/expired approvals, and requests with an invalid Origin.
5. Test conversations, sessions, and state isolation; check Persian text, code, and keyboard interaction.
6. Run real DeepSeek smoke tests only in an authorized environment; do not describe mock results as proof of web service compatibility.

A failed gate must stop the update, not be addressed by removing tests or hiding errors. Identify the previous working version and its lockfile for rollback; rollback is not a substitute for fixing incompatible changes. The fork owner is responsible for PR review until another maintainer is designated.

Revisit the [architecture decision](architecture-decision.md) when changing architectural direction or selecting a library version. This policy does not guarantee stability of DeepSeek's private web API; transport repairs may still be needed.