# Documentation Validation Report

Review date: `2026-10-28`; baseline version: `0.10.1`. This report records the original snapshot from the preparation of the `doc/` collection, not release approval or complete product testing. Its historical metrics do not assert that the current documentation count is unchanged.

## Scope and Method

The analysis covered entry-point code, the agent, tools, web transport, the server, the terminal interface, settings, prompts, and existing tests. The [coverage map](../reference/source-map.md) connects Git-tracked files to their documentation. Evidence for each behavior is linked on its dedicated page.

In the original snapshot, only Markdown files were added to `doc/`; build output, application code, the lockfile, root documents, and personal configuration were not edited. Running example commands with side effects or requiring a DeepSeek account was not part of validation.

## Documentation Checks

| Check | Criterion |
| --- | --- |
| Link targets | All relative file links resolve to existing targets |
| Code fences | Fenced blocks are closed |
| Navigation | All Markdown pages in the collection are reachable from the index |
| Repository coverage | Every file in the initial Git inventory has a source-map reference |
| Scope restriction | The diff for code and previously tracked files is empty, and new changes are limited to doc |
| Readability | Text review checks purpose, topic structure, separate examples, and reference links |
| Claim validity | Derived behavior, static risks, and untested runtime behavior are not conflated |

Automated file-target checks do not establish correct visual Markdown rendering or the semantic correctness of every sentence. Links in this collection intentionally target files rather than relying on fragile heading or line anchors. Mermaid diagrams are text-based; rendering in every viewer was not tested in this work.

## Environment and Build Limitations

Node in the original review environment was `v24.17.0`. The command `pnpm exec tsc --noEmit` was attempted, but the shell stopped with `pnpm: command not found`; it therefore produced no TypeScript result.

Further inspection showed that `node_modules`, a local TypeScript compiler, and local AVA were also absent. No dependencies were installed, no package manager was downloaded, and no network requests were made to satisfy these prerequisites. As a result:

- Build and typecheck are not reported as executed and successful.
- Lint, Prettier, and AVA were not run.
- The count of 12 cases was extracted from test definitions; it does not mean 12 tests passed.
- The absence of editor errors does not establish a successful build.

[package.json](../../package.json) defines how `pnpm build` and `pnpm test` run. To perform these gates in a prepared environment, follow [development setup](setup-and-scripts.md) and [testing](testing.md). The [ignore file](../../.prettierignore) excludes Markdown; even a successful pnpm test does not replace link checks.

## Behaviors Not Exercised

No personal token was read or recorded. Remote accounts, response generation, actual DeepSeek service health, session deletion, function calls, model switching, actual resuming, shell execution, elevation, and symlink scenarios were not tested. The security scenarios in [known limitations](../reference/known-limitations.md) are inferences from code, not executed exploits.

## Original Snapshot Results

These values describe the original documentation snapshot only, not the current collection or this translation's validation results.

| Check result | Value in the original snapshot |
| --- | --- |
| Markdown pages | 28 |
| Relative file links checked | 586; no broken targets |
| Initial Git inventory coverage | 85 of 85 files |
| Pages unreachable from the index | 0 |
| Unclosed code fences | 0 |
| Previously tracked files changed | 0 |
| New files outside doc | 0 |
| Editor errors reported in doc | 0; not a substitute for a compiler |

These checks used Node and the local filesystem without installing libraries. Text review also cross-checked sensitive CLI references, function parsing, settings, modes, and transport limitations against the sources; this review is not a formal guarantee that every sentence is correct.

## Rerunning the Structural Check

From the repository root, this check only reads files and throws an exception on link, fence, navigation, or coverage errors. It requires neither pnpm nor a DeepSeek account:

```sh
node --input-type=module <<'NODE'
import fs from 'node:fs'
import path from 'node:path'
import {execFileSync} from 'node:child_process'

const pages = new Map()
function scan(directory) {
	for (const entry of fs.readdirSync(directory, {withFileTypes: true})) {
		const file = path.join(directory, entry.name)
		if (entry.isDirectory()) scan(file)
		else if (file.endsWith('.md')) pages.set(file, fs.readFileSync(file, 'utf8'))
	}
}
scan('doc')
const edges = new Map()
for (const [file, text] of pages) {
	if ((text.match(/^```/gm) ?? []).length % 2) throw Error(`Fence: ${file}`)
	const targets = []
	for (const match of text.matchAll(/\]\(([^)]+)\)/g)) {
		if (/^(https?:|mailto:)/.test(match[1])) continue
		const [relative, anchor] = match[1].split('#')
		if (anchor) throw Error(`Anchor needs separate validation: ${file}`)
		const target = path.normalize(path.join(path.dirname(file), decodeURI(relative)))
		if (!fs.existsSync(target)) throw Error(`Link: ${file} -> ${target}`)
		targets.push(target)
	}
	edges.set(file, targets)
}
const visited = new Set()
function visit(file) {
	if (visited.has(file)) return
	visited.add(file)
	for (const target of edges.get(file) ?? []) if (pages.has(target)) visit(target)
}
visit('doc/README.md')
for (const file of pages.keys()) if (!visited.has(file)) throw Error(`Orphan: ${file}`)
const tracked = execFileSync('git', ['ls-files', '-z'], {encoding: 'utf8'}).split('\0').filter(Boolean)
const inventory = new Set(edges.get('doc/reference/source-map.md'))
for (const file of tracked) {
	if (!file.startsWith('doc/') && !inventory.has(file)) throw Error(`Uncovered: ${file}`)
}
console.log({pages: pages.size, links: [...edges.values()].flat().length, status: 'OK'})
NODE
```

This checker is designed for ordinary file links and the fences in this collection, not as a complete Markdown parser. Adding anchors, HTML, images, or a new fence type requires extending the check accordingly. Checking the diff for change scope and reviewing the text's meaning remain separate requirements.

The date and version belong to the analysis snapshot; the documentation does not guarantee automatic synchronization with future versions. The [maintenance guide](documentation-maintenance.md) defines the criteria for the next review.