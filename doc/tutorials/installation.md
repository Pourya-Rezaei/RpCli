# Installation and Setup

This tutorial takes you from a Node.js environment to running `rc` interactively.
You need a terminal, a DeepSeek web account, and network access during use.

## 1. Check Prerequisites

```sh
node --version
npm --version
```

Node.js 22 or later is recommended for the locked dependencies.
The package declares a minimum of 18, but Ink 7 in the dependency lockfile requires Node.js 22.
This recommendation comes from inspecting dependencies, not from runtime compatibility testing of every version.

## 2. Install the Package

```sh
npm install --global @rezaparsian/rp-cli@0.10.1
rc --version
```

The expected version output is `0.10.1`; verify it in your environment.
These commands are instructions for the reader and were not run while preparing this document.
To build from the repository, see [development setup](../development/setup-and-scripts.md).

## 3. Configure the Web Token

```sh
rc
```

If `DEEPSEEK_TOKEN` is absent, the token setup form appears.
Sign in to your DeepSeek web account in a browser and inspect the command displayed by the application.
That command reads the web session token from `localStorage`; this is not an official API key.
Enter the token only in the application's masked input, not in a support request or Git.
If the browser does not open automatically, open the displayed address manually.
Saving the token does not mean it has been successfully validated by the server.

Settings are stored in `~/.config/rp-cli/.env`.
An existing environment variable is not overwritten when dotenv loads.
See [configuration and storage](../reference/configuration-and-storage.md) for precedence and permissions.

## Next Step and Evidence

Continue with your [first session](first-session.md); the [documentation index](../README.md) lists the available paths.
Entry point and options owner: [source/cli.tsx](../../source/cli.tsx), variables `cli` and `firstArg`.
Form owner: [source/components/TokenSetupView.tsx](../../source/components/TokenSetupView.tsx), `TokenSetupView`.
Storage owner: [source/core/TokenConfig.ts](../../source/core/TokenConfig.ts), `saveDeepSeekToken` and `tokenConfigPath`.
Baseline version: **0.10.1**; review: **2026-10-28**; method: static analysis, without live DeepSeek testing.