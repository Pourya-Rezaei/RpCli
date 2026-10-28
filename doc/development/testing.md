 # Running Tests and Understanding Existing Coverage

 This guide distinguishes between defined tests, executed tests, and demonstrated behavior.
 Before running commands, prepare the [development environment](setup-and-scripts.md).

 ## Commands to Run

 ```sh
 pnpm build
 pnpm test
 ```

 The build includes TypeScript compilation and prompt copying.
 The test command runs `prettier --check .`, `xo`, and `ava` in that order.
 If the first stage fails, this command provides no evidence that AVA succeeded.
 For a focused check, after installing dependencies you can run:

 ```sh
 pnpm exec tsc --noEmit
 pnpm exec ava test/tools.test.ts
 ```

 The AVA configuration includes the `ts` and `tsx` extensions and the `ts-node/esm` loader.
 These examples are execution instructions, not results from this review.

 ## Defined Coverage: 12 Cases

 Five cases are defined in [source/core/bidiText.test.ts](../../source/core/bidiText.test.ts).
 They check left-to-right text, Persian reordering, line independence, Persian detection, and bidi terminal detection.
 Three cases are defined in [source/core/toolCallMarkup.test.ts](../../source/core/toolCallMarkup.test.ts).
 They cover complete DSML, chunk boundaries, and the less-than sign in ordinary text.
 Four cases are defined in [test/tools.test.ts](../../test/tools.test.ts).
 They check reading a file range, fallback from thinking, search/edit batches, and hiding incomplete markup.
 Only the file-range case actually executes `read_file` in that suite.
 The case count is neither a coverage percentage nor a safety certification.

 ## Gaps to Consider When Making Changes

 There is currently no direct coverage for the server, DeepSeek transport, or session lifecycle.
 Cancellation, batch confirmation, modes, output paths, file writing, and todos also lack direct coverage.
 For a related change, add a focused test of that contract; do not rely on these 12 cases as general assurance.
 Design tests that need files to use temporary storage without user data.
 Do not require DeepSeek calls in unit tests; explicitly separate token and network requirements.

 Markdown is ignored by the formatter; `pnpm test` does not guarantee valid documentation links.
 Dependencies and pnpm were unavailable in the review environment; build, lint, and AVA were not run.
 This review supports no claim that tests passed or that any coverage percentage was achieved.

 ## Evidence and Validity

 Local execution owner: [source/tools/execute.ts](../../source/tools/execute.ts), `executeTool`.
 Stream transformation owner: [source/core/toolCallMarkup.ts](../../source/core/toolCallMarkup.ts), `createToolCallMarkupNormalizer`.
 For the exact contract, see the [tool protocol](../reference/tool-protocol.md) and [index](../README.md).
 Baseline version: **0.10.1**; review: **2026-10-28**; static analysis, without live DeepSeek testing.