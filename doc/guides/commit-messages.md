# Generating and Approving Commit Messages

This guide covers creating a message from a diff and reviewing it before running Git.
This workflow sends the diff content to DeepSeek; check for secrets first.

## Preparing Changes

In the target repository, review the status and diff yourself:

```sh
git status --short
git diff --staged
```

Stage the intended files using your usual workflow; this guide does not promise any automatic staging.
Then request a message generated from the staged changes:

```sh
rc -c
```

The application sends the message-generation instruction to the model first, followed by the diff.
Review the generated text for scope, accuracy, and sensitive information.
Conventional Commits is the prompt's target, not a format guaranteed by application validation.

## Reviewing Broader Changes

To select `git diff HEAD` as the message-generation input:

```sh
rc -c -a
```

In this application, `-a` does not stage any files.
It also does not add the `-a` option to `git commit`.
The message may therefore describe tracked, unstaged changes, while the commit includes only staged changes.
Untracked files are not included in `git diff HEAD` either.
Before approving, align the message input with the actual commit content.

## Approval and Result Verification

The `CommitView` interface requests `y/n` confirmation; approve only after reviewing.
The final operation is `git commit -F -`, with the message supplied through standard input.
Hooks, Git identity, and repository rules can still reject the commit.
If an error occurs, check Git status; do not treat message generation as a successful commit.
A request to delete the message-generation session is sent, but successful remote deletion is not guaranteed.

## Evidence and References

Diff and commit owner: [source/actions/commitMessage.ts](../../source/actions/commitMessage.ts), `getGitDiff` and `executeCommit`.
Generation owner: `generateCommitMessage` in the same file.
Approval owner: [source/components/CommitView.tsx](../../source/components/CommitView.tsx), `CommitView`.
Options appear in the [CLI reference](../reference/cli.md), and the model instruction appears in [prompts](../reference/prompts.md).
For general limitations, see [known limitations](../reference/known-limitations.md) and the [index](../README.md).
Baseline version: **0.10.1**; reviewed: **2026-10-28**; static analysis, without runtime testing of DeepSeek.