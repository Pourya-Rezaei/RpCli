# Your First Interactive Session

In this tutorial, you ask a low-risk question about a repository and retain the ID needed to resume the session.
First complete [installation](installation.md) and open a terminal in the project directory.
For this exercise, choose a repository without secrets or sensitive data.

## 1. Open the Chat

```sh
rc
```

A new session first sends the system prompt to the service; this step can use the network.
Enter `/help` to display the commands actually available in this version.
Keep `normal` mode for this exercise.
If you need to change modes, TAB cycles through `normal → yolo → plan`.

## 2. Make a Bounded Request

```text
Inspect the structure of the source directory and explain only the roles of three main files. Do not change any files.
```

The natural-language instruction above expresses your requested constraint; it does not replace runtime enforcement.
Reads may occur without separate confirmation.
For each proposed change, read the tool name and preview before confirming.
In the tool confirmation dialog, Enter also means approval; use `n` to reject.
An `@path` reference inserts only the path text and does not automatically attach file contents.

## 3. Evaluate the Response

Check file names against the repository and ask for evidence for important claims.
If the task reaches the tool-round limit, `/continue` sends a request to continue the work.
This command does not remove the limit; it starts a new turn.
For the planning workflow, read [interactive workflows](../guides/interactive-workflows.md).

## 4. Exit and Return

Exit with `/exit` and retain the ID shown in the exit message.
Then run the following with your actual ID:

```sh
rc resume SESSION_ID
```

The local transcript and remote history are different things; resuming does not guarantee a complete replay.
Disabling `/logging` does not prevent the interactive transcript from being saved.
See [resuming and model switching](../guides/resume-and-model-switching.md) and the [index](../README.md) for details.

## Evidence and Validity

Lifecycle owner: [source/components/chat/useChatSession.ts](../../source/components/chat/useChatSession.ts), `useChatSession`.
Control owner: [source/components/chat/useComposerKeys.ts](../../source/components/chat/useComposerKeys.ts), `useComposerKeys`.
Command owner: [source/commands/index.ts](../../source/commands/index.ts), `slashCommands`.
Baseline version: **0.10.1**; review: **2026-10-28**; static analysis, without live DeepSeek testing.