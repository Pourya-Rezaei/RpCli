# Prompt Reference

Version `0.10.1`, reviewed `2026-10-28`. These files instruct the model; they are not security controls or guarantees of output quality on their own.

## Builders and Assets

Loading owner: [source/prompts/index.ts](../../source/prompts/index.ts). `loadPrompt` synchronously reads a file adjacent to the module; a missing asset can cause an error when a builder is called.

| Builder | File | Usage |
| --- | --- | --- |
| `SystemPrompt` | [system.md](../../source/prompts/system.md) and [tools.md](../../source/prompts/tools.md) | Start a local-agent conversation |
| `ServerSystemPrompt` | [server.md](../../source/prompts/server.md) | Start a server session without enabling local-tool execution |
| `InitPrompt` | [init.md](../../source/prompts/init.md) | Request generation of an AGENTS guide for the repository |
| `ContinuePrompt` | [continue.md](../../source/prompts/continue.md) | Continue a task after stopping or reaching the tool limit |
| `GitCommitMessage` | [gitMessage.md](../../source/prompts/gitMessage.md) | Instruction to generate a commit message |
| `PlanPrompt(task)` | [plan.md](../../source/prompts/plan.md) | Insert the task into the planning instruction |
| `ExecutePlanPrompt(mode)` | [executePlan.md](../../source/prompts/executePlan.md) | Start an approved plan in the target mode |

## Context Injection

`SystemPrompt` substitutes platform, cwd, and tool descriptions. Tool descriptions come from `getToolDescriptions` in the registry. Elevation guidance is generated for win32 or other platforms as appropriate.

`readAgentsMarkdown` reads only the AGENTS file in cwd; it does not traverse the parent hierarchy or load separate guides in each subdirectory. A missing file becomes an empty string; errors reading an existing file are not necessarily suppressed. SystemPrompt and ServerSystemPrompt append available content to the prompt.

ServerSystemPrompt does not inject the local-tool registry list. HTTP request functions are added to the request prompt through `withToolInstructions`, and their execution is the client's responsibility.

## What Is Enforced?

| Concern | Actual Control Layer |
| --- | --- |
| Planning advice, appropriate responses, and Conventional Commits | Prompt text; no general structural guarantee |
| Blocking mutating tools in plan | Executor, independently of model compliance |
| Rejecting paths outside cwd in specific tools | Path helper, with documented exceptions |
| Operation confirmation | Interface callback and mode in the executor |
| Tool-round limit | getAIResponse |

A "read only" prompt must therefore not be treated as a sandbox. Generating an AGENTS file through `/init` also depends on the model and tools, rather than being deterministic scaffolding within the slash command.

## Build and Change Lifecycle

[scripts/copy-prompts.mjs](../../scripts/copy-prompts.mjs) copies only direct `.md` files to `dist/source/prompts/`. `pnpm build` runs tsc, then copy; `pnpm dev` copies once, then watches TypeScript. Editing Markdown does not necessarily trigger automatic recopying, and stale output is not removed.

When making changes, review the source asset, builder, and consumer together. After a tool-contract change, registry descriptions, format examples, and the parser must also remain aligned. The [Maintenance Guide](../development/documentation-maintenance.md) contains the documentation checklist.