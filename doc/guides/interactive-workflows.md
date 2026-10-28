# Interactive Work with Change Control

This guide covers investigating, planning, and implementing a scoped repository change.
First check Git status yourself and keep existing changes separate from the new request.

## Investigation and Planning

Run `rc` in your chosen working root; the working directory is the base for local tools.
Select `plan` mode with TAB; the mode cycle is `normal → yolo → plan`.
There is no `--plan` CLI option or `/plan`, `/normal`, or `/yolo` command for selecting a mode.
State the desired result, permitted files, and verification method in your request.

```text
Plan a fix for this behavior; inspect only source/core and propose test criteria.
```

In `plan`, file changes and shell execution are blocked in the tool-batch path.
Changes to some in-memory todos remain allowed; do not interpret "read-only" as the absence of all internal changes.
The planning prompt expresses intent and is distinct from tools' execution controls.

## Approval and Execution

After the plan response, Enter or `y` accepts it and sends a request to execute the plan.
The mode returns to the previous non-`plan` mode; before accepting, check that it was not `yolo`.
Esc or `n` rejects plan approval and leaves planning mode active.
In `normal`, tools marked as making changes require confirmation.
Read the change preview; Enter in a tool confirmation means approval.
Rejecting one tool also skips the rest of that batch; a single error does not necessarily stop the batch.

In `yolo`, most confirmations are skipped, but detecting a need for elevation still requires confirmation.
Do not use this mode for an unfamiliar project or an unbounded request.
`run_command` is a real shell, not a file or network sandbox.

## Continuation and Context Management

`/todos` displays the task list held in this process's memory.
`/continue` is useful for continuing a request after the tool-round limit.
`/clear` creates a fresh conversation; it does not clear todos or the entire previous display.
`/init` asks the model to generate repository guidance; do not assume it is a non-modifying operation.
`/search` and `/thinking` toggle their respective states; search is not supported for every model.
To stop generation, press Ctrl+C while loading; cancellation is not immediate for every network stage.

## References and Evidence

The complete table is in [keys and commands](../reference/keyboard-and-slash-commands.md).
Tool boundaries: [local tools](../reference/local-tools.md) and [known limitations](../reference/known-limitations.md).
Execution owner: [source/tools/execute.ts](../../source/tools/execute.ts), `executeToolCalls` and `toolRequiresConfirmation`.
Loop owner: [source/actions/agent.ts](../../source/actions/agent.ts), `getAIResponse`; each call is limited to 10 tool rounds.
Approval owner: [source/components/chat/PlanConfirmation.tsx](../../source/components/chat/PlanConfirmation.tsx), `PlanConfirmation`.
Return to the [index](../README.md).
Baseline version: **0.10.1**; reviewed: **2026-10-28**; static analysis, without runtime testing of DeepSeek.