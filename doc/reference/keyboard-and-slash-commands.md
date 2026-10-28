# Commands and Keyboard

Version `0.10.1`, code reviewed on `2026-10-28`; actual terminal behavior was not tested at this stage.

## Registered Commands

Owner: `slashCommands` and `resolveSlashCommand` in [commands/index.ts](../../source/commands/index.ts). Matching is exact after trimming and lowercasing; there is no general argument parser for slash commands.

| Command | Alias | Effect |
| --- | --- | --- |
| `/init` | None | Send the AGENTS-generation prompt and attempt to reload the file after the response |
| `/continue` | None | Send ContinuePrompt to continue previous work |
| `/clear` | None | Reset chat and delete the previous transcript, not the todos |
| `/help` | None | Display help generated from the command registry |
| `/search` | None | Toggle web search |
| `/thinking` | None | Toggle thinking |
| `/logging` | `/log` | Toggle diagnostic logging and its preference |
| `/todos` | None | Display the in-memory todo list |
| `/model` | None | Switch between default and expert through the interface confirmation flow |
| `/exit` | `/quit` | Request application exit |

`/plan`, `/normal`, and `/yolo` are not registered. `/clear` does not treat display history as clearing all messages; another logo is added and session state is reset. Command effects are implemented in `runCommand` in [InteractiveChatView.tsx](../../source/components/InteractiveChatView.tsx).

## Text Entry and Mode

Owners: [useComposerKeys.ts](../../source/components/chat/useComposerKeys.ts) and [textCursor.ts](../../source/core/textCursor.ts).

| Key | Effect in the Active Composer |
| --- | --- |
| TAB | Cycle normal -> yolo -> plan -> normal |
| Left/Right arrow | Move along grapheme boundaries |
| Ctrl or Meta with Left/Right | Move by word |
| Home / End | Start/end of line |
| Ctrl+Up / Ctrl+Down | Start/end of text |
| Up / Down | Adjacent line within multiline text |
| Backspace / Delete | Delete before/after the cursor |
| Ctrl with deletion | Delete by word; Backspace detection also depends on terminal encoding |

TAB does not necessarily cycle modes while a picker, loading state, or confirmation is active; the composer hook is disabled in those states. Input and submission are also affected by the textarea component and Ink; this table does not claim complete compatibility with every terminal emulator.

## Scrolling, Stopping, and Exiting

Owner: the general `useInput` handler in [InteractiveChatView.tsx](../../source/components/InteractiveChatView.tsx).

- PageUp and PageDown call `scrollBy(-1)` and `scrollBy(1)` when no picker is active; full-page jumps are not implemented.
- Ctrl+Home and Ctrl+End target the start and end of the chat display.
- Esc requests a stop while loading; if initialization has not succeeded, immediately stopping all upstream work is not guaranteed.
- Ctrl+C requests a stop while loading; when idle with existing text, it clears the text first.
- Ctrl+C with empty input and an existing session first displays the resume message; the next press requests exit with code 130.

## Pickers and Confirmation

[FzfFilePicker.tsx](../../source/components/FzfFilePicker.tsx) has an internal fuzzy matcher; an external fzf executable is not required. Selecting a file only inserts `@path` into the composer and does not automatically attach its contents. [SlashCommandPicker.tsx](../../source/components/SlashCommandPicker.tsx) selects registered commands.

The [ToolConfirmation.tsx](../../source/components/ToolConfirmation.tsx), [PlanConfirmation.tsx](../../source/components/chat/PlanConfirmation.tsx), and [ModelConfirmation.tsx](../../source/components/chat/ModelConfirmation.tsx) dialogs have separate contracts. Tools are approved with `y` or Enter and rejected with `n`; Enter approves here rather than providing a safe rejection default. A plan starts with Enter or y and is rejected with Esc or n; a model change is confirmed with Enter and canceled with Esc.

Accepting a plan restores the previous non-plan mode; if you were in yolo before plan, execution of the plan can also start in yolo. See [local-tools.md](local-tools.md) for the effects of modes on tools.