# Tauri fork

This branch carries a narrow Tauri macOS carrier over the `dsh-v0.2.1-rc.2` upstream DeepSeek Harness revision. Run `git merge-base HEAD upstream/master` to recover the base revision. The branch ports intent rather than replaying the earlier fork history.

## Patch inventory

- `apps/tauri` is a thin macOS system-webview shell over `dsh web`. Its adapter reuses the maintained Python SDK executable builder and runtime closure, then bundles the executable, node-pty spawn helper, and ripgrep companion under Tauri `externalBin` names. The [Tauri shell Agent Note](.agents/notes/implemented/architecture/2026-09-15-tauri-thin-web-shell.md) owns lifecycle and verification details.
- The `llm-pi-ai` adapter stamps the per-conversation `x-opencode-session` routing header on OpenCode routes, which the OpenCode gateways require and pi-ai does not yet send. The [OpenCode session header Agent Note](.agents/notes/implemented/feature/2026-09-17-opencode-session-header.md) owns the detail.
- Tauri releases use the independent `tauri-v<version>` tag family. `apps/tauri/src-tauri/tauri.conf.json` is the desktop version authority and Cargo matches it; the private npm workspace version remains repository-tooling metadata.
- Downstream CI runs on standard public GitHub-hosted runners with bounded worker counts while upstream keeps custom, self-hosted, and Blacksmith runner selection. The [downstream CI Agent Note](.agents/notes/implemented/process/2026-09-16-downstream-hosted-ci.md) records runner, post-merge, and live-API policy.

Authenticated non-loopback Web Host binding and trusted-remote settings persistence are no longer fork work: upstream `dsh-v0.2.1-rc.2` binds a concrete non-loopback `--host` natively (rejecting only wildcard addresses through `isWildcardHost`) and persists authenticated browser preferences through the Host (upstream PR #4201), covered by upstream's `apps/web/tests/remote-settings.e2e.ts`. The fork's superseded `feat(web): persist settings for trusted remote hosts` commit is dropped; restoring it would re-add behavior upstream now owns.

No package-wide version changes, legacy sidecar closure, packaged CLI bootstrap, UUID fallback, VFS workaround, annotation feature, Herdr skill, or release-family rewrite belongs to this fork layer.

## Upstream integration checklist

1. Re-apply the fork feature commits (Tauri shell, Tauri remote-connect fix, OpenCode session header) and the downstream CI overlay onto the selected upstream revision, resolving behavior by current APIs rather than restoring deleted packages or generated catalogs. Confirm upstream has not absorbed an item before re-applying it.
2. Confirm `scripts/build-tauri-sidecar.ts` remains only an adapter over `scripts/build-exe-for-python-sdk.ts`, passes the `--targets` spec the current builder accepts, and copies every required executable companion.
3. Run the source Tauri smoke, supervisor tests, the `llm-pi-ai` adapter tests, scoped static checks, and documentation synchronization. Run the signed bundle and normal-Quit smoke on macOS before publication.
4. Compare the Tauri configuration and Cargo versions before creating a `tauri-v` tag. Do not infer the desktop version from the private npm manifest or root release family.
5. Recheck every owner-qualified runner selector, setup condition, worker budget, and live-API caller after upstream workflow changes; keep both `main` and `master` in post-merge triggers.

## Coordination

The sibling `refactory-dsh` repository consumes the harness as a published npm dependency and tracks its own move onto `dsh-v0.2.1-rc.2` ([refactory-dsh#276](https://github.com/boston-harness-group/refactory-dsh/issues/276)). The two are coupled only through the operator: this checkout is the harness the local tooling runs, so a divergent pin would leave two harness versions on one machine. This fork moved to `dsh-v0.2.1-rc.2`; `refactory-dsh` is reconciled separately.

## Durable data

Tests set fresh temporary `DSH_HOME`, `DSH_AGENTS_HOME`, credential inputs, and workspace paths; they never inspect or mutate a user's Harness home. Current Session format v3 retains the released adjacent migration chain: an accepted v0 log is read into a new version-named successor while the predecessor remains unchanged. Rebase verification uses the committed v0 migration fixture under temporary storage rather than real user data.

## Stage 1 checkpoint

The preserved `stage1/macos-bundle-regressions` branch passed 7 cross-platform supervisor tests, the full source runtime smoke, the insecure-HTTP browser replay, build and scoped checks. macOS Actions run `35010702817` additionally passed signed bundle inventory, frontend/RPC/settings/image/PTY/descendant cleanup, and actual application normal-Quit cleanup. The branches returned by `git branch --list 'safety/*'` retain the original baseline.
