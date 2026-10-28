# Terminals, Persian Text, and Display Direction

Review date: 2026-10-28 | Version: 0.10.1
This explanation is source-based; layout in real terminals, fonts, and Persian input have not been tested at runtime.

## Logical Text Versus Display Text

The storage order of Persian characters is not the same as the order of visible cells on the screen.
In text mixed with paths, numbers, and Latin identifiers, the bidirectional algorithm determines display order.
Do not manually reverse the original message content or file path to fix an unsuitable display.
`toTerminalText` in [bidiText.ts](../../source/core/bidiText.ts) addresses this distinction at the display boundary.
Its design intent is to keep API text and stored data in logical order and transform only terminal-facing strings.
This function is not a letter-shaping engine, font selector, or complete implementation of cursor behavior.

## RTL Detection and Terminal Capability

`containsRightToLeftText` detects Hebrew, Arabic, and related presentation-form ranges using a regex.
This detection relies on the presence of characters, not linguistic analysis or identifying a sentence as Persian.
`terminalSupportsBidirectionalText` considers the presence of `VTE_VERSION` or `KONSOLE_VERSION` sufficient.
This is an environmental heuristic, not a practical test of terminal behavior or a correctness guarantee for every version.
If these variables are present, `toTerminalText` returns the text unchanged.
In other environments, the function uses `bidi-js` for each line containing RTL text.
`getEmbeddingLevels` computes direction levels, and `getReorderedString` builds the display string.
Lines without RTL characters remain unchanged, and newline separation is preserved.
An incorrectly set environment variable may therefore be inconsistent with actual display behavior.

```text
logical message -> Markdown tokens -> terminal-facing transformation -> Ink
logical message -> transcript / API (not a visual reverse of the message)
```

## Markdown in Ink

`MarkdownText` in [MarkdownText.tsx](../../source/components/MarkdownText.tsx) uses Marked tokens and Ink components.
This path is not a browser DOM or an HTML/CSS engine with a direction property.
`terminalToken` and `terminalCell` apply direction transformation to text and token children.
Headings, paragraphs, emphasis, links, basic lists, tables, and code blocks have specific rendering paths.
The presence of Marked does not mean that terminal output fully supports every Markdown detail.
For example, lists use simple bullets; browser output appearance should not be treated as the terminal contract.
`renderInline` places link text and its address alongside each other in text output.
Combining a Latin address with a Persian title is a direction-sensitive display case.

## Block Code Versus Inline Code

`terminalToken` returns early for a token of type `code` and does not reorder its direction.
Code blocks are displayed through `SyntaxHighlight` so general reordering does not alter code text.
`syntaxLanguage` converts aliases such as `ts` and `sh` and falls back to `plaintext` for an unknown language.
By contrast, `codespan` is not covered by the code-block exception, so direction transformation may apply to its text.
Putting Persian text inside inline backticks therefore does not offer the same guarantee of passing through without reordering as a block.
When reviewing a complex mixed-text command or path, assess the source's logical text separately from its visual appearance.
This document does not recommend adding hidden direction-control characters to executable commands.

## Tables and Text Width

In `renderBlock`, table column widths are calculated using string length and padding.
JavaScript string length is not necessarily the number of terminal cells; combining and wide characters can differ.
Bidirectional reordering alone does not resolve this measurement difference.
A table containing Persian, combining marks, or mixed text therefore needs inspection in the target terminal.
Table support in the renderer should not be interpreted as a guarantee of alignment in every font and window width.
The same distinction matters for paragraph wrapping and long-path readability.

## Input and Cursor Coordinates

[textCursor.ts](../../source/core/textCursor.ts) converts the input string into lines, columns, and offsets.
`endPosition`, `cursorOffset`, and `positionAt` rely on newlines and string length.
These coordinates are logical string positions, not direct measurements of visual columns after BiDi.
`previousCharacterOffset` and `nextCharacterOffset` use `Intl.Segmenter` with `grapheme` granularity.
This choice avoids reducing each movement to a single UTF-16 unit and matters for multi-character clusters.
However, segmentation alone does not determine the visual direction of left/right keys or Persian shaping.
`previousWordOffset`, `nextWordOffset`, and `nextDeleteWordOffset` rely on whitespace patterns.
Here, "word" is therefore a practical, whitespace-based definition, not Persian lexical analysis.
The Segmenter's `en` locale should not be taken as a claim that Unicode text is disabled; the selected unit here is a grapheme.

## Ownership of Keys and Suggestions

`useComposerKeys` in [useComposerKeys.ts](../../source/components/chat/useComposerKeys.ts) makes decisions among input, pickers, and conversation mode.
`mentionQuery` in [textCursor.ts](../../source/core/textCursor.ts) detects an `@` expression at the end of input.
`slashCommandQuery` extracts the start of a slash command from input.
`FzfFilePicker` in [FzfFilePicker.tsx](../../source/components/FzfFilePicker.tsx) uses internal fuzzy matching and returns at most seven results.
Selecting a path only inserts a textual mention into the composer; file content is not automatically attached.
`SlashCommandPicker` in [SlashCommandPicker.tsx](../../source/components/SlashCommandPicker.tsx) displays command suggestions.
The actual command set is defined by `slashCommands` in [commands/index.ts](../../source/commands/index.ts), not by names appearing in a prompt.
Key details should be read in the [keyboard and command reference](../reference/keyboard-and-slash-commands.md).

## Live Streaming and Display Stability

`createLiveStream` in [createLiveStream.ts](../../source/components/chat/createLiveStream.ts) batches incoming stream text for display at roughly 50-millisecond intervals.
As a sentence becomes complete, mixed-text direction levels may change; an incomplete chunk does not always represent the final layout.
`MessageRow` in [MessageRow.tsx](../../source/components/chat/MessageRow.tsx) and the main view separate message roles from growing content.
Hiding tool markup is a separate path and should not be confused with RTL transformation.
`ChatStatusBar` in [ChatStatusBar.tsx](../../source/components/chat/ChatStatusBar.tsx) displays interface state; it is not an indicator of correct Persian rendering.
Dependence on Ink and the terminal's key encoding makes the actual result environment-dependent.

## Test Evidence and Confidence Limits

[bidiText.test.ts](../../source/core/bidiText.test.ts) declares five AVA cases for direction-transformation logic.
These cases were not run as part of this work and do not establish coverage of all terminals, letter shaping, or cursor behavior.
A future review should separately examine pure Persian text, Persian with Latin paths, numbers, zero-width non-joiners, and combining characters.
It should also compare VTE/Konsole environments with environments lacking those variables, multiline input, and window-width changes.
That review would aim to compare stored text with logical text and independently assess display appearance, not reverse data for visual appeal.
No visual result from that hypothetical review is reported here as completed.

## Related Reading

[Architecture](architecture.md) clarifies component ownership, and [execution flows](execution-flows.md) explains the relationship between streams and interface messages.
[Configuration and storage](../reference/configuration-and-storage.md) records transcript locations, and [known limitations](../reference/known-limitations.md) defines the bounds of compatibility claims.
For the complete reading path, see the [documentation index](../README.md).