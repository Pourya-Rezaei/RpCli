# Setting Up the Development Environment

This guide is for developers building a checked-out copy of the repository.
The commands below are for later use in a prepared environment; they were not run during this review.

## Prerequisites and Installation

Choose Node.js 22+ for the locked dependencies; the package declares a minimum of 18.
The presence of Node.js 24 in the review environment does not demonstrate that the project was tested on it.
The manifest does not pin a pnpm version through `packageManager`.
Having Corepack installed does not mean pnpm is ready to run.

```sh
pnpm install
pnpm build
```

Installing dependencies requires the appropriate permissions and network conditions in your environment.
pnpm and dependencies were absent from the documentation environment; no installation or download was performed.
Build output goes to `dist`; do not edit its files manually.

## Build and Watch

`pnpm build` runs `tsc` first, then copies the prompts.
The actual destination for prompt Markdown files is `dist/source/prompts`.
The script copies only top-level `.md` files from the prompt directory.
This process does not completely clean old output.

```sh
pnpm dev
```

Dev mode copies prompts once and then runs `tsc --watch`.
Subsequent edits to prompt Markdown are not automatically copied again; rerun the build or copy step.
After building, run locally with:

```sh
node dist/source/cli.js
```

This invocation may request a token and connect to DeepSeek; do not treat it as an offline check.
`DEEPSEEK_API_KEY` is not part of this CLI's contract; it uses a web token as described in [installation](../tutorials/installation.md).

## Quality Checks and Evidence

`pnpm test` runs Prettier, XO, and AVA in sequence; see the [testing guide](testing.md).
`pnpm format` formats the entire repository; do not run it unnecessarily for a narrow change.
CLI routing owner: [source/cli.tsx](../../source/cli.tsx), `cli`.
Prompt loading owner: [source/prompts/index.ts](../../source/prompts/index.ts), `SystemPrompt` and `ServerSystemPrompt`.
For the symbol map, see [exported symbols](../reference/exported-symbols.md) and the [index](../README.md).
Baseline version: **0.10.1**; review: **2026-10-28**; static analysis, without a build or live DeepSeek testing.