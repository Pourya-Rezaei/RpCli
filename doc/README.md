# RpCli Documentation

This collection explains how to understand, use, and extend RpCli: a DeepSeek chat CLI, terminal interface, local-tool agent, and HTTP adapter. It documents traceable implementation behavior rather than repeating a product introduction.

**Baseline version:** `0.10.1`, from [package.json](../package.json). **Analysis date:** `2026-10-28`.

## Evidence and Scope

Source analysis is distinct from runtime testing. Behavioral descriptions point to linked files and symbols; examples requiring DeepSeek access or file mutations were not executed during authoring. See the [verification report](development/verification.md) for checks and unavailable build prerequisites, and [known limitations](reference/known-limitations.md) for constraints and discrepancies in older documentation.

Documentation changes are confined to `doc/`. Application code, the [root README](../README.md), [AGENTS.md](../AGENTS.md), personal settings, tokens, and build output were not modified. Consult source evidence where older descriptions conflict with the baseline implementation.

## Reading Paths

| Reader goal | Suggested sequence |
| --- | --- |
| First use | Installation, first session, interactive workflows |
| Understand internals | Architecture, execution flows, DeepSeek transport, source map |
| Build an HTTP client | Server guide, API reference, limitations and security |
| Modify the application | Development setup, interfaces, extension guide, testing and maintenance |
| Assess retained data | Configuration and storage, security, session resume |

## Tutorials

- [Installation and prerequisites](tutorials/installation.md)
- [First session](tutorials/first-session.md)

## Task Guides

- [Interactive workflows](guides/interactive-workflows.md)
- [Session resume and model switching](guides/resume-and-model-switching.md)
- [Commit message generation](guides/commit-messages.md)
- [Local HTTP server](guides/http-server.md)
- [Troubleshooting](guides/troubleshooting.md)

## Technical References

- [CLI syntax and routing precedence](reference/cli.md)
- [Configuration and data storage](reference/configuration-and-storage.md)
- [Keyboard controls and slash commands](reference/keyboard-and-slash-commands.md)
- [Local tools and execution policies](reference/local-tools.md)
- [Tool-call protocols](reference/tool-protocol.md)
- [HTTP contract and compatibility matrix](reference/http-api.md)
- [Prompts and project context](reference/prompts.md)
- [Important interfaces and exported symbols](reference/exported-symbols.md)
- [Known limitations and documentation discrepancies](reference/known-limitations.md)
- [Repository source coverage map](reference/source-map.md)

## Architecture Explanations

- [Layers and module ownership](explanation/architecture.md)
- [Execution flows](explanation/execution-flows.md)
- [DeepSeek transport, PoW, and SSE](explanation/deepseek-transport.md)
- [Trust boundaries, security, and privacy](explanation/security-and-privacy.md)
- [Terminal and Persian text rendering](explanation/terminal-and-persian-rendering.md)

## Development and Quality

- [Proposed web interface and architectural decisions](web-development/README.md). This is a future design, not an implemented feature.
- [Development setup and scripts](development/setup-and-scripts.md)
- [Existing tests and coverage gaps](development/testing.md)
- [Extending tools and commands](development/extending-tools-and-commands.md)
- [Documentation maintenance](development/documentation-maintenance.md)
- [Documentation verification report](development/verification.md)

## Shared Terminology

| Term | Meaning in this project |
| --- | --- |
| workspace / cwd | Process launch directory; the basis for file tools and AGENTS context |
| Remote session | A conversation in the DeepSeek account with a session ID and parent message |
| Transcript | Local interactive messages used to restore the resume display; independent of logging |
| Log | Optional diagnostic records of prompts, events, and results |
| Generation | A response-production turn with shared cancellation state |
| Tool round | Receiving calls, executing tools, and returning results to the model |
| Continuation | Extending an incomplete upstream response; distinct from tool rounds and /continue |
| normal / yolo / plan | Interface tool-execution policies, not DeepSeek model names |
| default / expert | Interactive model selection, distinct from HTTP model labels |
| Client function | An HTTP-declared function executed by the client, not the local registry |
| Prompt | Textual instructions to the model, not proof of runtime enforcement |

## Readability and Maintenance

Each page has one purpose. Reference pages own contract tables; tutorials and guides link to them. Documentation prose, examples, and paths use English. Multiline examples use code fences. Discussion of Persian rendering describes an application capability without requiring Persian text in the documentation.

Source-confirmed behavior is not the same as tested behavior, and a static risk is not an exploit-confirmed vulnerability. Future changes must review the date, evidence, affected contract, and limitations together. Use the [maintenance guide](development/documentation-maintenance.md) and [source map](reference/source-map.md) to track that work.