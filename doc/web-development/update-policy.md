# Update and Compatibility Policy

Date: `2026-10-28`. Status: target maintenance policy; no update automation has been configured by this documentation change.

## Independent Upstreams

| Project | Update path | Compatibility responsibility |
| --- | --- | --- |
| RpCli | Fetch the verified original repository through an upstream remote and review merges in the fork | Preserve existing CLI and DeepSeek behavior plus the integration contract |
| Open WebUI | Upgrade its independently installed release or image | Check connection configuration, plugins, persistence, migrations, and protocol behavior |
| Optional adapter/plugin | Maintain small integration code against documented versioned contracts | Revalidate chat identity, stream, cancellation, and approvals |

Keeping updates available does not mean fetching latest at runtime or merging without review. Fixes reach users only after the chosen versions have been tested and deployed.

## RpCli Maintenance

Keep origin pointing to the owner's fork. Verify the original repository URL and branch before adding an upstream remote; this document does not configure one. Review upstream changes on a separate integration branch with a clean worktree. Resolve conflicts without dropping local work, then run existing and new regression gates.

Do not force-push shared branches or rewrite history without permission. Keep integration additions separate from original modules where practical. Record the reason for each necessary core patch so an upstream fix can later replace it. Contributing back to the original project requires a separate PR, not an automatic push.

## Open WebUI Maintenance

Prefer an unmodified pinned release. Do not vendor its Svelte frontend or maintain a source fork merely to change the chat appearance. A supported extension or external adapter is preferable when configuration cannot express the required contract.

Pin a release version or immutable image digest for reproducibility. Do not assume tags named main or latest mean a stable release. Review the selected release's notes, runtime requirements, connection settings, plugin interfaces, and security advisories before upgrading.

Back up the database and file storage before an update. Use separate data directories/volumes for evaluation and normal use. Preserve required instance secrets securely. Database migrations may prevent a simple binary/image downgrade; rollback needs a compatible backup, not just the old executable.

An extension that works with one release is not automatically compatible with the next. Document the tested RpCli/Open WebUI/adapter version combination and installation method. Keep update PRs or deployment changes separate from unrelated feature work to identify failures.

## Licensing and Branding

Open WebUI is not covered solely by a simple MIT license. Its current license includes branding conditions and exceptions; prior components may retain their original licenses. Check the [LICENSE](https://github.com/open-webui/open-webui/blob/main/LICENSE) and [LICENSE_HISTORY](https://github.com/open-webui/open-webui/blob/main/LICENSE_HISTORY) for the exact version being used or redistributed.

Retain upstream branding by default. Do not promise unrestricted rebranding, redistribute a modified package, or assume today's personal-use circumstances authorize a future public product. Review licensing before redistribution or branding changes. This policy is an engineering requirement, not legal advice.

## Update Gates

1. Build/typecheck and run the existing RpCli formatting, lint, and AVA gates in a prepared environment.
2. Exercise terminal, single prompt, commit, resume, and serve without unexpected changes.
3. Verify model discovery, request conversion, non-stream responses, fragmented streaming, errors, and cancellation with a controlled mock service.
4. Check chat identity and auxiliary model requests; run New Chat and history tests for implemented milestones.
5. For implemented file milestones, test allowed/denied paths, approval rejection, replay, disconnects, and unchanged user files after denial.
6. Run browser acceptance tests against the actual pinned Open WebUI release, then a separately authorized real-DeepSeek smoke test.

Automated dependency PRs may later use one chosen tool such as Dependabot or Renovate for repository-owned dependencies. Such a bot does not automatically manage an independently deployed application's database and backups. Do not enable unattended deployment merely to receive fixes sooner.

If a gate fails, stop the upgrade and retain the tested configuration. Do not delete tests to pass. Record unverified areas explicitly, including upstream private-web-API changes that may require transport fixes. See the [delivery plan](delivery-plan.md) for staged capability boundaries.