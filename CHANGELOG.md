# Changelog

All notable changes to the public **Mindustry-UX** branch are recorded here. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and versions use prerelease SemVer tags (`v0.1.x-alpha.N`).

This file covers only changes made by this fork. Upstream Mindustry changes are not repeated; refer to [Anuken/Mindustry releases](https://github.com/Anuken/Mindustry/releases). Experimental work stays in the private development repository and is not listed until it is exported and validated here.

## [Unreleased]

## [0.1.0-alpha.1] - 2026-11-22

First public prerelease. Built from source commit [`67f0b478dd`](https://github.com/WaterOfbigRiver/Mindustry-UX/commit/67f0b478ddcc0ab58823b0c426276531e61586b5), whose parent is the audited upstream commit `19a0f40d44514f4802a8b79770456e7e32e34da0`.

### Added

- **Campaign wave timer is enabled on world load.** Loading a campaign world now sets `state.rules.waveTimer = true`, so sectors and older saves that store `waveTimer=false` show a real countdown instead of a permanent "waiting" HUD state. Non-campaign games are unaffected.
- **Explicit sector save actions.** A sector with an existing save now offers separate **Resume** and **New Game** actions. Starting a new game asks for confirmation, copies the current sector save to a single backup slot, and aborts without deleting the current save if copying or metadata writing fails.
- **Restore previous save.** The backup can be restored after confirmation. The backup file is `<planet>-s-<id>-backup.msav` with metadata key `<planet>-s-<id>-info-backup`; only one backup is kept per sector, and backup files are hidden from normal save slots.
- Regression tests in `tests/src/test/java/ApplicationTests.java` covering the wave timer rule and the reset/restore save flow.
- English, Simplified Chinese, and Traditional Chinese bundle strings for the new actions and confirmations.
- Release assets: `Mindustry.jar` (desktop, Java 17), `server-release.jar`, `SHA256SUMS.txt`, release notes, and two screenshots captured from this public branch.
- `README.md` describing the fork scope, differences from upstream, backup semantics, build and test steps, known limitations, and the GPLv3 modification notice.

### Changed

- Default build type is `Mindustry-UX` instead of `official`, so modified builds do not identify themselves as official Mindustry.
- CI is read-only: upstream push/deploy steps that wrote to Arc, MindustryGame/docs, or MindustryBuilds were removed, and the deployment workflow is a manual read-only build.
- The Arc dependency is pinned to commit `7445105cd2683daf5b9ac7033ebb07265d3e814d` instead of tracking a moving branch.
- The public branch starts from an audited upstream commit; it is not the private branch with files removed, and no private commit is an ancestor of it.

### Not included

- No Android APK is published. The public branch does not change `applicationId` or signing, so a later APK release needs a separate compatibility and signing decision.
- Experimental developer UI and startup themes, RTS-style mouse controls, unlimited core storage, extended tutorials and objective cards, event log/HUD reworks, mobile pause-menu redesigns, internal documents, and local configuration remain private.
- The upstream `abandonSectorConfirm` flow and its behavior are unchanged.

### Verified

- `./gradlew tests:cleanTest tests:test` from a fresh single-branch clone: 349 tests, 0 failures, 0 errors.
- Desktop `Mindustry.jar`: 88,516,853 bytes; server `server-release.jar`: 19,302,955 bytes.
- `version.properties`: `type=Mindustry-UX`, `build=1`, `modifier=alpha`, `commitHash=67f0b478dd`.
- Gitleaks scan of the final tree and full public history: only pre-existing upstream false positives (a Serbian keybind label and the public upstream Discord client ID).

### Known limitations

- This is a prerelease; save compatibility with later versions is not guaranteed. Back up game data before use.
- The desktop JAR requires Java 17. Other JDK versions are outside the tested baseline.
- Issue reports are limited to the public changes above; bugs that also occur in official Mindustry belong upstream.

[Unreleased]: https://github.com/WaterOfbigRiver/Mindustry-UX/compare/v0.1.0-alpha.1...public-master
[0.1.0-alpha.1]: https://github.com/WaterOfbigRiver/Mindustry-UX/releases/tag/v0.1.0-alpha.1
