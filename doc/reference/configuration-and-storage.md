# Configuration and Storage

Review basis: version `0.10.1` on `2026-10-28`; code inspection without examining the user's account or personal files.

## Configuration Sources

[cli.tsx](../../source/cli.tsx) and [server/index.ts](../../source/server/index.ts) load the configuration file with dotenv; `override` is not requested. Existing environment values therefore generally take precedence over the same names in the file. The model preference is an exception because it is read directly from the file.

| Name | Consumer | Meaning and Default |
| --- | --- | --- |
| `DEEPSEEK_TOKEN` | App, API client, server | DeepSeek web-session token; required for sending |
| `RC_TOKEN` | Server `getToken` | Fallback upstream token; not an independent client password |
| `DEEPSEEK_SESSION_ID` | API client at import | Initial session ID; reset by new interactive initialization |
| `DEEPSEEK_MESSAGE_ID` | API client at import | Initial numeric parent-message ID |
| `RP_CLI_LOG_CHAT` | LogChat | Enabled with `1`, `true`, or `yes`; off by default |
| `RP_CLI_MODEL` | TokenConfig | Stored `default` or `expert`; other values use default |
| `PORT` | startServer | Default 3000; CLI option takes precedence |
| `HOST` | startServer | Default 127.0.0.1; CLI option takes precedence |

`DEEPSEEK_API_KEY` does not replace `DEEPSEEK_TOKEN` in this implementation. The HTTP model name also differs from the interactive model preference.

## Configuration File

Owner: [TokenConfig.ts](../../source/core/TokenConfig.ts), symbols `tokenConfigDirectory` and `tokenConfigPath`.

Fixed path relative to home on every platform:

```text
~/.config/rp-cli/.env
```

`XDG_CONFIG_HOME` is not used to relocate this path. `saveDeepSeekToken`, `clearDeepSeekToken`, `saveChatLoggingPreference`, and `saveModelPreference` rewrite values in a Map; existing comments and formatting are not preserved. Removing the last entry deletes the file.

`getModelPreference` reads the file directly; merely setting `RP_CLI_MODEL` in the environment does not necessarily change the interface's model selection. `expert` is the only accepted non-default value for this preference.

New directories request mode `0700`, and files request `0600`. These creation requests do not repair permissions on existing files or directories and do not encrypt data.

## Three Independent Stores

| Store | Location/Owner | Retained Content | Depends on Logging |
| --- | --- | --- | --- |
| Diagnostic log | Under `logs/<session-id>/` in config; LogChat | Prompts, events, and raw results | Yes |
| Interactive transcript | `sessions/<session-id>.json` in config; SessionTranscript | User, thinking, assistant, and console; no logo | No |
| Service session | DeepSeek account; core-lib | Remote history and state | No |

Evidence: [LogChat.ts](../../source/core/LogChat.ts), [SessionTranscript.ts](../../source/core/SessionTranscript.ts), and [useChatSession.ts](../../source/components/chat/useChatSession.ts).

Turning off `/logging` does not necessarily stop interactive transcript storage. Raw logs and transcripts serve different purposes and do not share an automatic retention policy. Local deletion does not prove remote deletion.

`SessionTranscript` restricts IDs to letters, digits, `_`, and `-`. Stored text retains its original logical order; RTL transformation concerns display. Logging errors are mostly ignored without stopping chat; a missing log file does not always mean no request was made.

## Lifecycle and Token Precedence

On the server, selection order is `DEEPSEEK_TOKEN`, then `RC_TOKEN`, then the request's Bearer token. When an environment token exists, the client's Bearer token is not compared for independent authentication. Details are in [http.ts](../../source/server/http.ts).

An unused interactive session is submitted for best-effort remote deletion on exit; a used session is retained for resume. `/clear` deletes the old transcript and resets the chat session, not every store or the todos. Switching models after a conversation creates a new session with a summary.

`deleteSession` only guarantees receipt of a fetch response; the wrapper does not assert HTTP success. For data-retention purposes, completion of `await` must not be equated with deletion of information from the service.

## Safe Handling

Do not publish real tokens, the config directory, or transcripts in the repository or bug reports. Tool workspace paths do not isolate the application's home configuration file from shell access. See [Security and Privacy](../explanation/security-and-privacy.md) for exact boundaries and [Troubleshooting](../guides/troubleshooting.md) for diagnosis.