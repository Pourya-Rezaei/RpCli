# Development Rules and Compatibility Boundaries

Date: `2026-10-28`; status: proposed implementation requirements for [decision WEB-001](architecture-decision.md). "Must" indicates an acceptance criterion, not a claim about the current implementation.

## Preserve Existing Behavior

1. CLI, prompt, commit, resume, and serve must have regression tests; removing the old interface or changing the `rc` default requires separate approval.
2. Do not replace the DeepSeek transport or credentials with another service or official SDK; consume the existing connection.
3. Do not copy agent or tool logic. Reuse it through a narrow internal API; unavoidable core changes must include rationale and tests.
4. New web endpoints must use a separate namespace; do not change existing public endpoint envelopes or semantics without authorization.
5. Keep frontend packages and configuration as separate as possible from terminal TypeScript and builds. Importing browser dependencies must not force old paths to load the DOM.

## Integration Contract

Exact endpoint names will be determined later; required behavior is already defined:

| Concept | Requirement |
| --- | --- |
| Run | Unique identifier and explicit status; one active run in the first release |
| Message | Stable identifier and ordering; text and reasoning can be displayed separately |
| Stream | Distinguishable events for text, tools, awaiting approval, completion, cancellation, and errors |
| Approval | Operation identifiers bound to the run and workspace; reject reuse or approval of another operation |
| Cancellation | Propagated from the browser to that same run; late callbacks must not attach to a new run |
| Disconnection | Pending approval must not be accepted automatically; define a timeout or safe cancellation |
| History | Explicit UI/remote mapping; selecting a new conversation must not leak into the previous session |

SSE or the final transport must have an appropriate parser; do not treat a network chunk as a complete event. The adapter must follow the selected runtime's format; do not confuse cumulative state with deltas. AI SDK streams, OpenAI SSE, and the internal protocol are not interchangeable.

## Approval and Security

- Tool access must occur only in Node; the browser requests operations but does not create operating-system permissions.
- Loopback is the baseline, but is insufficient: Host/Origin validation, a restrictive CORS policy, and local session access authentication are required.
- Local bridge credentials must be separate from the upstream token; do not expose secrets in URLs, logs, localStorage, or the bundle. Design the bootstrap method before implementation.
- The server controls mode, workspace, and operation permissions; browser-supplied data alone is not authoritative.
- Message text, Markdown, and tool output are untrusted; do not execute raw HTML or active content without a safe policy.
- Token input must be masked, and API responses must never return the stored token.
- File and shell tools retain their existing limitations; an attractive interface does not create a sandbox. Account for [current risks](../explanation/security-and-privacy.md) in bridge design.

## Local Lifecycle

The launch cwd must be the fixed workspace; the browser may not arbitrarily select system paths without a permission contract. The local listener and workspace must belong to the same instance. If the port is occupied, select a free port and print a usable address; the exact command syntax will be determined in launcher design.

Ctrl+C and exit signals must clean up pending approvals, runs, and the listener. Closing a tab is not the same as ending the Node process; specify disconnection and session retention behavior separately. Include interface assets in the release package instead of downloading them on each run.

## User Experience

Persian text must reach the browser in its original logical order; do not apply terminal-specific bidi transformations to web text. Test mixed-direction text and code blocks with RTL/LTR cases. Do not present unfinished controls: disable or omit upload, regenerate, and edit when backend support is absent.

Stop and approval buttons must reflect actual server status. Keyboard navigation, visible focus, accessible control names, and mobile behavior are acceptance requirements. Plan and normal modes must remain aligned with actual executor policy; do not enable web yolo in the first release without an explicit decision.

## Code Change Rules

Keep changes focused and small; avoid unrelated cleanup, generated-file changes, and broad reformatting. Keep the registry, parser, and tool descriptions synchronized when contracts change. Review affected code, tests, and documentation as one logical change. Bridge API changes must also have a clear version or compatibility policy.

Do not accept changes claiming that "updates will never conflict". The correct criteria are limited dependencies, tests, and a review process. The [update policy](update-policy.md) and [delivery criteria](delivery-plan.md) provide complementary requirements.