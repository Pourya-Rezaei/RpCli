# Resuming Sessions and Switching Models

This guide covers continuing previous work or transferring context to another model.
Treat the session ID as sensitive operational data and do not publish it alongside conversation content.

## Resuming a Session

Keep the ID shown in the exit message for a used session, and run this in the same project:

```sh
rc resume SESSION_ID
```

Replace `SESSION_ID` with the actual ID; without it, the CLI exits with an error.
Starting a resume selects the remote ID and loads any existing local transcript.
This path does not resend the system prompt as it would for a fresh session.
The absence of a local transcript does not necessarily mean that the remote session is absent.
If the remote session is not found, the next send may create a fresh session; strict resumption is not guaranteed.
To diagnose this, inspect the response's session ID and the context of the model's response.

## Three Distinct Data Locations

The interactive transcript is stored in `~/.config/rp-cli/sessions/<session-id>.json`.
This transcript retains user, response, thinking, and console messages, but not logos.
Optional logs in `~/.config/rp-cli/logs/<session-id>/` contain events and more raw data.
`/logging` or `/log` changes only this optional logging.
Turning it off does not disable the interactive transcript or delete existing data.
DeepSeek history is the third location and is distinct from local files.
For configuration precedence and permissions, read [storage](../reference/configuration-and-storage.md).

## Switching Models While Preserving Context

Enter `/model` in the interactive interface and review the corresponding confirmation.
The models in this interface are `default` and `expert`.
After user activity, the transfer summarizes the conversation and creates a fresh session.
This transfer is not a full copy of the messages; restate important details in your next request.
The previously used remote session remains.
The model preference is stored in the configuration file; merely setting the `RP_CLI_MODEL` environment variable is not equivalent.
Web search for a non-default model is restricted in the underlying transport.

## Cleanup and Limitations

`/clear` restarts remote conversation state and deletes the previous local transcript.
Do not treat it as a command to clear todos or as a guarantee that all remote data is deleted.
On exit, deletion of a fresh, unused session is requested on a best-effort basis.
Receiving a response to a deletion request is not proof of successful remote deletion.
For errors, see [troubleshooting](troubleshooting.md) and the [index](../README.md).

## Evidence and Validity

Transfer owner: [source/components/chat/useChatSession.ts](../../source/components/chat/useChatSession.ts), `useChatSession`.
API owner: [source/core/apiClient.ts](../../source/core/apiClient.ts), `setCurrentSessionId` and `sendMessage`.
Transcript owner: [source/core/SessionTranscript.ts](../../source/core/SessionTranscript.ts), `loadSessionTranscript` and `saveSessionTranscript`.
Baseline version: **0.10.1**; reviewed: **2026-10-28**; static analysis, without runtime testing of DeepSeek.