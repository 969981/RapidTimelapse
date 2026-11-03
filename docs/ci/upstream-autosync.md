# RapidTimelapse upstream sync and multi-platform releases

## Verified upstream
- Official upstream: https://github.com/crispin81/RapidTimelapse
- Upstream branch: master, initial SHA 17ae2344e365d70173675b0b5b1c108f35e2f5b3 (verified on 2026-11-03).
- Fork default branch: master. The fork was identical to official master when automation was added.
- Existing upstream workflows are preserved: .github/workflows/release.yml (manual / v* tags) and build-linux.yml.

## Automation
- .github/workflows/sync-upstream.yml: scheduled every 3 hours at minute 41 UTC, manually dispatchable, and bootstraps on first push of its YAML.
- Fetches official master and merges new commits into fork master through a real Git merge (without auto-resolving conflicts). This keeps the fork-specific automation files intact and does not force-push away user history.
- If a merge conflicts, CI fails for manual resolution. An upstream history rewrite detected against the last dispatched SHA is refused.
- The state ref rapidtimelapse-autosync-state points to the *last dispatched* upstream SHA (not last successful build).
- New upstream SHA -> dispatch build-release.yml for that exact SHA, otherwise skip. No tag conflicts with the official v* release workflow.

## Building and releasing
- build-release.yml creates a **draft** prerelease autosync-<12-character upstream SHA>.
- Uses the verified official .github/workflows/release.yml native packaging recipe and tauri-apps/tauri-action@v0 with three runner jobs: Linux ubuntu-22.04, Windows latest, and macOS latest with universal-apple-darwin for Intel + ARM.
- Linux packages: .AppImage, .deb, .rpm; Windows: .msi and .exe; macOS: universal .dmg.
- A Rust build cache (Swatinem/rust-cache) and npm dependency cache reduce repeated compile time.
- The release remains a **draft** until every job succeeds AND the final job confirms expected installers. On failure, keep it as a draft for investigation, and manually re-run the failed run with the same SHA.
- The Tauri bundles are **unsigned**; Windows SmartScreen and macOS Gatekeeper may display warnings.
- This is a development snapshot; it is not the upstream official signed stable build.

## Troubleshooting
- GitHub Actions must allow GITHUB_TOKEN write permissions for contents and actions; build and release jobs require contents write.
- New SHA -> one dispatch, not automatic retry for failed compilations. Retry manually in the Actions workflow with the full 40-char SHA.
- Schedule times are approximate. Linux AppImage build relies on an upstream modified GTK deploy hook.
- If an upstream source change also edits sync files or causes a merge conflict, review it in the PR/merge failure rather than force-overwriting.
- Windows, Linux and macOS native builds consume Actions minutes on the relevant hosted runner pools.

## Links
- Actions: https://github.com/969981/RapidTimelapse/actions
- Releases: https://github.com/969981/RapidTimelapse/releases
- Upstream reference: https://github.com/crispin81/RapidTimelapse/blob/master/.github/workflows/release.yml
