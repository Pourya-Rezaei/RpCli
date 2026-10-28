# Command-Line Reference

This document describes the application's input contract from the code; it is not a step-by-step guide.

## Source and Scope

Review basis: [source/cli.tsx](../../source/cli.tsx), [source/app.tsx](../../source/app.tsx), and [package.json](../../package.json), version `0.10.1`, reviewed `2026-10-28`. The behaviors on this page were derived from code; a live DeepSeek connection has not been tested.

## Invocation Forms

| Input | Result |
| --- | --- |
| `rc` | Interactive chat |
| `rc "prompt"` | Send a single request and exit |
| `rc resume <session-id>` | Enter interactive chat with a previous session ID |
| `rc -c` | Generate a commit message from staged changes and request confirmation |
| `rc -c -a` | Generate a message based on `git diff HEAD` |
| `rc serve` | Start the HTTP server without loading the Ink interface |

The first argument `serve` is checked before other modes are selected. On the non-server path, `-c` takes precedence over the prompt. Text arguments are joined with spaces and trimmed. `resume` without an ID exits with an error message and exit code `1`.

## Options

| Option | Short | Type | Purpose |
| --- | --- | --- | --- |
| `--commit-message` | `-c` | boolean | Select the commit-message generation view |
| `--commit-all` | `-a` | boolean | Change the diff input in commit mode; does not stage files |
| `--thinking` | `-t` | boolean | Enable thinking for a single request |
| `--quiet` | `-q` | boolean | Hide thinking output for a single request |
| `--search` | `-s` | boolean | Enable web search for a single request |
| `--port` | `-p` | string | Server port; converted with `parseInt` |
| `--host` | None | string | Server bind address |
| `--help` | None | Meow built-in | Display help |
| `--version` | None | Meow built-in | Display version |

Boolean flags become `false` when absent when passed to App. Interactive mode has its own internal settings; do not assume that `-t` and `-s` change its defaults. The server defaults to `127.0.0.1:3000`; environment precedence and validation details are in the server implementation.

## Examples That Request No File Changes

```sh
rc --help
rc --version
rc "Explain this project without changing files"
rc -tq "Explain the tradeoffs briefly"
rc serve --host 127.0.0.1 --port 8080
```

The two prompt examples run in the agent environment despite requesting no changes in their text; prompt text does not replace execution restrictions or tool-confirmation review. The server example remains running until the process is stopped.

## Reliable Details

- `--plan` is not defined in the flags. Interactive mode selection uses TAB.
- `quiet` does not mean JSON output or guarantee clean stdout; no separate machine-piping contract is defined.
- `-a` does not include untracked files in `git diff HEAD` and does not commit them.
- Run the built file from the root with `node dist/source/cli.js`; `dist` is not an editable source directory.
- The application's working directory is the basis for the workspace and for reading `AGENTS.md`; the package installation location is not the user's project root.
