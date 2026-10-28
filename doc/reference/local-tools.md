# Local Tools Reference

Baseline version `0.10.1`; reviewed `2026-10-28`. The following contract comes from reading code, not performing operations on user files.

## Ownership and Execution Model

Tool definitions are in [registry.ts](../../source/tools/registry.ts), contracts in [types.ts](../../source/tools/types.ts), input conversion in [arguments.ts](../../source/tools/arguments.ts), and execution coordination in [execute.ts](../../source/tools/execute.ts). `tools` contains 12 tools. `getToolDescriptions` generates textual descriptions for the prompt; there is no central JSON Schema for validating every tool.

## Files, Search, and Shell

| Tool | Parameters | Behavior and Output | Confirmation in normal |
| --- | --- | --- | --- |
| `list_directory` | `path?`, default `.` | Direct entries labeled file or directory; nonrecursive | No |
| `read_file` | `path`, `offset?`, `limit?` | UTF-8 text; offset is one-based and defaults to 1; limit is the line count | No |
| `write_file` | `path`, `content` | Create or overwrite completely; output is the byte count | Yes |
| `edit_file` | `path`, `old_text`, `new_text` | Replace exactly one unique, exact match | Yes |
| `delete_file` | `path` | Delete a regular file, not a directory | Yes |
| `search_files` | `query`, `path?`, default `.` | At most 50 lines in path:line:text format | No |
| `run_command` | `command` | Execute shell; stdout and stderr result | Yes |

`read_file` rejects nonregular files and sizes above `100 * 1024` bytes; the range is calculated after reading the entire file. offset and limit must be positive integers. A range beyond the end of a file may return empty text.

`edit_file` treats zero or multiple matches as errors. Empty `old_text` is invalid; `new_text` may be empty. `write_file` does not create missing parent directories. The 100 KiB read limit does not imply the same limit for writing or editing.

`search_files` performs case-sensitive literal search, not regex search. Files larger than 100 KiB are skipped. File discovery begins in [ListWorkspaceFiles.ts](../../source/core/ListWorkspaceFiles.ts): Git first, then a fallback based on [Gitignore.ts](../../source/core/Gitignore.ts). This filter is not a confidentiality control; explicitly reading an ignored file remains possible.

## Todo List

Owner: [todo.ts](../../source/tools/todo.ts), symbols `todoTools` and `getTodoList`.

| Tool | Parameters | Effect |
| --- | --- | --- |
| `todo_add` | `description`, `parent_id?` | Create a pending task; adding a child to a done parent is rejected |
| `todo_list` | None | Display the list and completion percentage |
| `todo_update` | `id`, `status` | status is pending, in-progress, or done |
| `todo_split` | `id`, `subtasks` | Replace direct children and return the parent to pending |
| `todo_clear` | None | Clear the Map and reset the next ID to 1 |

All todo outputs are formatted text; even if a description mentions "returning an ID," the output contract is not structured JSON. Data is held only in process memory and is not restored with the session transcript.

`subtasks` accepts an array or string; strings are split by line, or by commas for a single line containing commas. A single item over 200 characters is rejected. Parent completion only examines direct children and is not general recursive propagation. Display also shows roots and one child level.

Only `todo_clear` has `requiresConfirmation: true`; the others are classified as nonmutating by this executor despite changing memory. They can therefore also run in plan mode. The issue with invoking parameterless tools through markup is explained in the [Tool Protocol](tool-protocol.md).

## Batch Policy and Confirmation

- `executeToolCalls` runs an entirely nonmutating batch in parallel; result order is preserved.
- A single mutating tool makes the entire batch execute sequentially.
- Tools classified as mutating are blocked in plan mode.
- yolo skips ordinary confirmations; elevation still requires confirmation.
- A tool error generally does not stop the next tool; rejecting confirmation skips the remainder of the batch.
- `executeTool` alone does not enforce mode or confirmation policy; that is the caller's responsibility.

The general `ToolResult` contains `ok`, `tool_name`, and either `result` on success or `error` on failure. Arrays become newline-separated text, and shell output becomes text with a stderr section. An error does not throw a raw exception to the model; it becomes an unsuccessful result.

## Safety Boundary

[paths.ts](../../source/tools/paths.ts) captures the initial cwd. `safePath` checks existing paths and realpaths; `safeTargetPath` resolves only the parent for write destinations. Final destination symlinks and search paths have separate limitations; details are in [Security](../explanation/security-and-privacy.md).

Ordinary shell execution uses the initial cwd, inherited environment, and a one-MiB buffer; it has no timeout or sandbox. The elevation path in [sudo.ts](../../source/tools/sudo.ts) uses textual `sudo` detection and `@slosk/sudo-prompt`, not a real shell parser. User confirmation does not create an operating-system-level restriction.

Read the [API Reference](http-api.md) for the distinction between these tools and HTTP functions.