# Adding a Local Tool or Slash Command

This guide identifies the change points; read the linked references for the complete contract.
First decide whether you need a model-callable tool or a direct user-interface command.

## Local Tools

The `Tool` contract includes `name`, `description`, `execute`, and optional `requiresConfirmation`.
Keep the implementation within the boundary defined by [source/tools/types.ts](../../source/tools/types.ts).
Add the tool to `tools` in [source/tools/registry.ts](../../source/tools/registry.ts).
`getToolDescriptions` generates prompt descriptions from this registry.
This contract has no machine-readable parameter schema; descriptions and runtime validation are separate.
Keep types, defaults, ranges, and error behavior consistent between both.
Use the validation pattern in [source/tools/arguments.ts](../../source/tools/arguments.ts).

For file operations, inspect the path pattern in [source/tools/paths.ts](../../source/tools/paths.ts).
`safePath` and `safeTargetPath` are not equivalent; the latter resolves the target's parent.
For an existing symlink target, checking only the parent does not guarantee confinement.
A mutating tool needs confirmation metadata and an appropriate preview.
Previews are built in [source/tools/describe.ts](../../source/tools/describe.ts).
Mode and confirmation controls live in `executeToolCalls`, not in `executeTool` itself.
Calling `executeTool` directly bypasses these controls; do not present it as a generally safe execution path.

Align names and parameters with the current parser.
The parser requires at least one parameter for an invoke, even if the tool description requires no arguments.
For the exact format, read the [tool protocol](../reference/tool-protocol.md).
HTTP client tools are separate from this registry, and the server does not execute them locally.

## Slash Commands

Definitions are owned by [source/commands/index.ts](../../source/commands/index.ts).
`SlashCommand` takes a name, optional alias, description, and `execute` function.
Pass UI dependencies through `SlashCommandContext`; wire the context into the consumer as needed.
Add the command to `slashCommands` so help and the resolver use the same definition.
`resolveSlashCommand` matches the entire trimmed, lowercased text; it is not a general argument parser.
For parameterized commands, explicitly account for and test this limitation.
Do not document existing mode changes using invented commands; the current mechanism is TAB.

## Pre-Delivery Checks

Write focused tests for invalid arguments, errors, and the successful path.
For mutating tools, also check `plan`, `normal`, and `yolo` behavior and rejection of confirmation.
After changing TypeScript, run a build or typecheck, followed by the relevant test.
Update the registry description, reference, and consumer example together.
Ready-to-run commands are in [testing](testing.md), and the behavior table is in [local tools](../reference/local-tools.md).

## Evidence and Validity

Execution owner: [source/tools/execute.ts](../../source/tools/execute.ts), `executeToolCalls` and `toolIsMutating`.
Parsing owner: [source/tools/parse.ts](../../source/tools/parse.ts), `parseToolCalls`.
The development map is in [exported symbols](../reference/exported-symbols.md) and the [index](../README.md).
Baseline version: **0.10.1**; review: **2026-10-28**; static analysis, without live DeepSeek testing.