# Execution Agent Handoff

Created: `2026-10-28`. Status: prepared for a new session; no implementation task has started.

This directory is the operational handoff for connecting Open WebUI to RpCli. It tracks execution, evidence, and review. The [architecture decision](../web-development/architecture-decision.md) remains authoritative for design; the [delivery plan](../web-development/delivery-plan.md) defines milestones. This directory does not create a competing architecture.

## Read in This Order

1. Repository [AGENTS.md](../../AGENTS.md) and applicable local instructions.
2. [Integration overview](../web-development/README.md), decision, rules, update policy, and delivery plan.
3. [Execution instructions](instructions.md).
4. [Checklist and authorization gates](checklist.md).
5. [Current status and work log](progress.md).

All handoff records must be in English. Preserve unrelated existing changes. Do not infer that a plan, checkbox, command example, or earlier conversation proves that work was performed.

## New Session Prompt

```text
Read doc/agent-handoff/README.md and every document listed in its reading order.
Inspect the actual worktree; preserve existing edits and do not assume prior chat memory.
Execute the checklist in order, starting at the first authorized incomplete task.
Only read-only reconnaissance is initially authorized. Record missing permissions and
environment decisions; request approval before installation, services, or implementation.
After each meaningful task, update doc/agent-handoff/progress.md with completed work,
current work, next work, changed files, checks, results, and blockers.
Mark a checkbox complete only after its acceptance evidence is recorded.
Continue automatically only to an authorized task within the same approved gate.
Stop at owner approval gates, unresolved contradictions, failures, or scope changes.
Do not build a custom frontend or replace RpCli's DeepSeek transport or agent.
Do not change existing CLI behavior, add state packages, commit, or push without approval.
Finish each session with an accurate handoff and leave no task falsely marked complete.
```

## Owner Supervision

Review [progress](progress.md) for the latest snapshot and append-only evidence. Use [checklist](checklist.md) to approve the next gate. An agent's completion claim and the owner's acceptance are separate states. Reports provide checkpoints, not guaranteed real-time monitoring; use small tasks and explicit gates to limit unnoticed mistakes.

Each approval must record its date, scope, constraints, and a reference to the owner's instruction. The agent may transcribe actual approval but must not grant it to itself. No commits, pushes, real credentials, or runtime installations have been authorized merely by creating this handoff.