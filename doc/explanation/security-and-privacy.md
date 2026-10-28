# Trust Boundaries, Security, and Privacy

Review date: 2026-10-28 | Version: 0.10.1
This is a static code assessment; no exploitation, penetration testing, or runtime testing was performed.
The risks below are "possibilities based on code paths", not vulnerabilities confirmed by an exploit.

## Assets and Trust Boundaries

The main assets are the web session token, conversation text, workspace files, process environment, and authority to execute shell commands.
User input, project file text, model responses, and HTTP requests do not have equivalent trust sources.
`getAIResponse` in [agent.ts](../../source/actions/agent.ts) brings model text toward a tool request.
This point requires execution controls because model output does not become trusted merely by appearing inside a valid tag.
`parseToolCalls` in [parse.ts](../../source/tools/parse.ts) is a parser; successful parsing is not proof of safety.
`arguments.ts` in [arguments.ts](../../source/tools/arguments.ts) validates and converts arguments, but does not replace system-wide policy.

## Prompt Intent Versus Execution Controls

`SystemPrompt`, `PlanPrompt`, and `ServerSystemPrompt` in [prompts/index.ts](../../source/prompts/index.ts) describe desired model behavior.
`readAgentsMarkdown` adds the text of `AGENTS.md` at the cwd root to context; this file is contextual input, not an operating-system authorization authority.
A project file or tool result may contain conflicting instructions; a prompt alone does not establish that their effects are prevented.
By contrast, `executeToolCalls` in [execute.ts](../../source/tools/execute.ts) actually checks mode and confirmation before executing a batch.
Calling `executeTool` directly bypasses this mode and confirmation path; developers should not treat it as equivalent to the policy-enforcing interface.
Adding a new tool requires reviewing metadata, validation, and the execution path together.

## Confirmation and Plan Scope

In normal mode, tools marked with `requiresConfirmation` require approval, not every read or every possible operation.
In yolo mode, ordinary confirmation is skipped; elevation detection still has a confirmation path.
In plan mode, classified modifications are blocked, but some in-memory todo changes remain allowed.
This classification is not equivalent to a sandbox or full semantic analysis of shell commands.
`ToolConfirmation` in [ToolConfirmation.tsx](../../source/components/ToolConfirmation.tsx) treats Enter as approval and accepts lowercase `y` and `n`.
The preview in [describe.ts](../../source/tools/describe.ts) and [diff.ts](../../source/tools/diff.ts) supports human decisions; it is not a general-purpose diff or proof that side effects are absent.
Rejecting confirmation can stop the rest of the batch; one tool's error does not necessarily prevent the next tool from running.

## File Path Boundary

`safePath` in [paths.ts](../../source/tools/paths.ts) checks both lexical path containment and realpath against the recorded cwd.
For read paths, this control rejects symbolic links pointing outside the boundary.
`safeTargetPath` resolves only the destination's parent with realpath; an existing final component does not necessarily receive the same check.
This difference suggests a static risk of redirecting `write_file` through a final symlink to an outside destination.
The normal-mode preview may reject the path earlier; direct execution or yolo does not guarantee the same preview path.
Writing outside the workspace was not actually tested in this review.
The `startsWith('..')` condition also over-restricts some valid names; this issue is separate from path escape.
Given the shell and the differing controls, saying "all application operations are confined to cwd" is not an accurate claim.

## File Discovery, Ignore Rules, and Confidentiality

`ListWorkspaceFiles` in [ListWorkspaceFiles.ts](../../source/core/ListWorkspaceFiles.ts) first uses Git's listing, with a traversal fallback.
[Gitignore.ts](../../source/core/Gitignore.ts) applies only a subset of ignore rules in the fallback.
Ignore rules are not a confidentiality boundary; explicitly reading an ignored path may be allowed.
`list_directory` likewise does not apply a general ignore filter.
In `search_files`, paths listed by Git are read without a separate `safePath` check for each file.
Following a listed symlink is therefore a potential risk of reading beyond the boundary, not confirmed exploited behavior.
File-size and result-count limits are useful for resource control, but do not replace sensitive-data classification.
The limits contract appears in [local tools](../reference/local-tools.md).

## Shell and Privilege Elevation

The `run_command` implementation in [registry.ts](../../source/tools/registry.ts) uses shell `exec` with the initial cwd and inherited environment.
This path has no command allowlist, network restriction, file sandbox, or timeout; its output buffer limit is 1 MiB.
Approving a command can therefore authorize access far beyond the file tools.
[sudo.ts](../../source/tools/sudo.ts) detects the word `sudo` with a regex; this detection is not a shell parser.
Even quoted text can trigger detection, and removing words/options and changing whitespace can alter command meaning.
The elevation path sends a filtered environment, but lacks the normal path's explicit cwd, AbortSignal, and timeout.
The cancel button should not be interpreted as a guarantee of stopping an elevated command or undoing changes.

## Credentials and Three Data Storage Locations

[TokenConfig.ts](../../source/core/TokenConfig.ts) stores configuration in `~/.config/rp-cli/.env`; this path follows the same pattern on all platforms.
Directory and file creation request modes `0700` and `0600`, respectively; existing permissions are not explicitly repaired.
`TokenSetupView`'s masked input reduces token visibility, but is neither file encryption nor remote validation before saving.
`RP_CLI_LOG_CHAT` controls optional raw-event logging in [LogChat.ts](../../source/core/LogChat.ts).
Independently, [SessionTranscript.ts](../../source/core/SessionTranscript.ts) retains the interactive transcript for resume.
Disabling logging therefore does not disable the local transcript or delete DeepSeek history.
Thinking messages, tool output, paths, and user text may be sensitive within retained data.
Transcript file IDs are restricted to allowed characters; this path control is distinct from remote session ownership or confidentiality.
`deleteSession` in [DeleteSession.ts](../../core-lib/DeleteSession.ts) does not turn a resolved fetch alone into proof of deletion.
The remote service's actual retention policy cannot be inferred from this code.

## A Local Server, Not a Multitenant Service

`startServer` in [server/index.ts](../../source/server/index.ts) serves HTTP; loopback should remain the preferred deployment boundary.
[http.ts](../../source/server/http.ts) provides wildcard CORS and private-network permission, not independent client authentication.
Token precedence is `DEEPSEEK_TOKEN`, then `RC_TOKEN`, then the request's Bearer token.
With a configured token, the requester need not provide the same credential; the upstream secret is not a separate client access key.
TLS, rate limiting, session ownership checks, and tenant isolation are not implemented at this layer.
`enqueueCompletion` in [session.ts](../../source/server/session.ts) has a shared queue; a queue does not imply security isolation between clients.
Tool-cache discovery and prompt preparation also occur before queued execution and must be considered in concurrency analysis.
Exposing the port beyond the host requires external protection and an independent access design; this document does not certify the safety of any particular reverse proxy.

## Practical Implications

Do not automatically assume ignored files are confidential, and review a shell command's effects independently of the model's explanation before approving it.
Include session tokens, transcripts, and logs when reviewing workspace sharing; the examples in these documents contain no real credentials.
These recommendations do not replace path-control, cancellation, and multi-client tests; those tests were not run here.
For contracts, see [configuration and storage](../reference/configuration-and-storage.md), [HTTP API](../reference/http-api.md), and [prompts](../reference/prompts.md).
For open issues, read [known limitations](../reference/known-limitations.md); for the discussion's context, read [architecture](architecture.md) and the [index](../README.md).