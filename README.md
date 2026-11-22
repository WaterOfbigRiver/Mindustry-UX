# Mindustry-UX

An **unofficial**, campaign-focused modified version of [Anuken/Mindustry](https://github.com/Anuken/Mindustry), maintained by WaterOfbigRiver.

This repository is not affiliated with, endorsed by, or supported by Anuken or the official Mindustry project. It is a fork of Mindustry that keeps the upstream GPLv3 license and adds a small, independently validated set of campaign UX changes.

- Public branch: `public-master` (default). `master` still tracks upstream and is not used for releases.
- Private development repository: `WaterOfbigRiver/Mindustry-UX-dev` (full history and experimental work).

## Screenshots

Sector actions shown for a sector that already has a save:

![Sector resume and new game actions](docs/images/campaign-sector-actions.png)

Campaign gameplay in this public prerelease:

![Campaign gameplay in the public prerelease](docs/images/campaign-gameplay.png)

Both images are captured from builds of this public branch and contain no player accounts, local paths, or private saves.

## What is different from upstream

The public version contains **two behavior changes** plus the supporting strings, tests, and CI changes. Everything else is upstream Mindustry.

### 1. Campaign wave timer is always enabled on world load

Upstream loads a sector's `waveTimer` rule from the map/save. Some campaign maps and older saves store `waveTimer=false`, which leaves the HUD showing a permanent "waiting" state instead of a countdown.

This version forces the automatic wave timer on when a campaign world is loaded:

```java
// core/src/mindustry/core/Logic.java, WorldLoadEvent
state.rules.waveTimer = true;
```

Effect:

| Situation | Upstream | Mindustry-UX |
|---|---|---|
| Sector map stores `waveTimer=false` | HUD shows "waiting", no countdown | Countdown runs normally |
| Sector map stores `waveTimer=true` | Countdown runs | Unchanged |
| Custom game / non-campaign | Rule is left alone | Unchanged |

### 2. Explicit "Resume / New Game" and previous-run recovery for sector saves

Upstream shows a single play action for a sector that already has a save. Starting over silently replaces the previous run.

This version:

| Action | Behavior |
|---|---|
| **Resume** | Loads the existing sector save, exactly as before |
| **New Game** | Confirms first, copies the current sector save to an explicit backup, then starts a fresh sector |
| **Restore Previous Save** | Confirms first, restores the backup over the current sector save |

Backup details:

- Backup file: `sector-<planet>-<id>-backup.msav` in the sector save directory.
- Backup metadata key: `<planet>-s-<id>-info-backup`, storing the sector info snapshot taken at backup time.
- Exactly **one** explicit backup is kept per sector; starting another new game replaces it.
- `Saves.load()` ignores files whose name contains `backup`, so the backup never appears as a normal save slot.
- If copying the save or writing the backup metadata fails, the new game is aborted and the current save is **not** deleted.
- On restore, `hasCore` is set back to `true` so the restored sector loads as a normal continuation instead of a lost-sector restart.
- Restore reuses the existing save slot when present, otherwise creates one, so no duplicate slots appear.
- The upstream `abandonSectorConfirm` flow is untouched.

### Files touched relative to upstream

| File | Purpose |
|---|---|
| `core/src/mindustry/core/Logic.java` | Enable the wave timer on campaign world load |
| `core/src/mindustry/game/Saves.java` | Backup file/meta helpers, reset and restore operations |
| `core/src/mindustry/ui/dialogs/PlanetDialog.java` | Resume / New Game actions, restore entry, error handling |
| `core/assets/bundles/bundle.properties` | New English strings |
| `core/assets/bundles/bundle_zh_CN.properties` | New Simplified Chinese strings |
| `core/assets/bundles/bundle_zh_TW.properties` | New Traditional Chinese strings |
| `tests/src/test/java/ApplicationTests.java` | Regression tests for both changes |
| `build.gradle` | Default build type is `Mindustry-UX`, not `official` |
| `.github/workflows/*.yml` | Read-only CI, pinned Arc commit, manual release build |

### Deliberately not included

Experimental developer UI and startup themes, RTS-style mouse changes, unlimited core storage, extended tutorials and objective cards, event log/HUD reworks, mobile pause-menu redesigns, internal documents, local configuration, and any unvalidated feature stay in the private development repository. The public branch is not a build of the private branch with files removed; it is a fresh change set on top of an audited upstream commit.

## Download

See [Releases](https://github.com/WaterOfbigRiver/Mindustry-UX/releases). Each release lists the exact source commit used for its JAR files and provides SHA-256 checksums. Fork changes per version are listed in [CHANGELOG.md](CHANGELOG.md).

`v0.1.0-alpha.2` is a prerelease providing:

- `Mindustry.jar` — desktop build (Java 17 required)
- `server-release.jar` — headless server
- `Mindustry-UX-v0.1.0-alpha.2-windows-x64.zip` — Windows x64 app image with a bundled Java runtime, launched from `Mindustry-UX.exe`; the executable is unsigned, so SmartScreen may warn
- `Mindustry-UX-v0.1.0-alpha.2-android-debug.apk` — **debug-signed** Android build for testing only; it keeps the official `io.anuke.mindustry` application ID, so uninstall the official app first
- `SHA256SUMS.txt` — checksums for the published files

Back up your game data before trying a prerelease; save compatibility with later versions is not guaranteed. The Android APK is a debug build, not a release build, and is not suitable for distribution or long-term use.

## Build from source

Use **JDK 17**. Other JDK versions are not supported by this build baseline.

```bash
git clone --branch public-master --single-branch https://github.com/WaterOfbigRiver/Mindustry-UX.git
cd Mindustry-UX
./gradlew desktop:dist server:dist
```

On Windows, use `gradlew` instead of `./gradlew`.

Outputs:

- `desktop/build/libs/Mindustry.jar`
- `server/build/libs/server-release.jar`

The Gradle wrapper is pinned to **Gradle 8.7**, which is the tested version for both the desktop build and the Android build.

Run the automated tests with:

```bash
./gradlew tests:test
```

Run the desktop client locally:

```bash
./gradlew desktop:run
```

Build an Android debug APK with an installed Android SDK (`compileSdk 36`, `build-tools 36.0.0`):

```bash
ANDROID_HOME=/path/to/android-sdk ./gradlew android:assembleDebug \
  -PversionType=Mindustry-UX -PversionModifier=alpha -Pbuildversion=2
```

The APK is written to `android/build/outputs/apk/debug/android-debug.apk`. It is debug-signed and keeps the official `io.anuke.mindustry` application ID, so it cannot be installed alongside or over an official build.

Builds used for a release are produced with an explicit, non-official version type:

```bash
./gradlew desktop:dist server:dist \
  -PversionType=Mindustry-UX -PversionModifier=alpha -Pbuildversion=2 -PshowCommitHash
```

The resulting `version.properties` reports `type=Mindustry-UX`, so a modified build never identifies itself as an official Mindustry build.

The Gradle dependency is pinned through `archash` (`7445105cd2`); CI fetches Arc commit `7445105cd2683daf5b9ac7033ebb07265d3e814d` explicitly instead of tracking Arc's moving `master` branch.

## Verification

`v0.1.0-alpha.2` uses the same game code as `v0.1.0-alpha.1` and adds packaging and build fixes:

- `./gradlew tests:cleanTest tests:test` — 349 tests, 0 failures, 0 errors
- `./gradlew android:assembleDebug` — debug-signed APK, package `io.anuke.mindustry`
- Windows package: the release `Mindustry.jar` is packaged with `jpackage` by the manual `Windows EXE` workflow; the JAR inside the package matches the released checksum
- Secret scanning (Gitleaks) over the final tree and the full public history: only pre-existing upstream false positives (a Serbian keybind label and the public upstream Discord client ID)

## Releases and tags

This fork uses its own `v0.1.x` prerelease tags. GitHub workflows do not push to Anuken/Arc, MindustryGame/docs, MindustryBuilds, or any other external repository. Release assets are built and reviewed before publication.

## Contributing and issue scope

Issues and pull requests here should concern the public Mindustry-UX changes. Report upstream-only problems to the official Mindustry repository. Do not attach private saves, credentials, tokens, signing keys, or personal paths.

## License and modification notice

The original [GNU General Public License version 3](LICENSE) is retained verbatim. Mindustry-UX modifications made on **2026-11-22** are distributed under the same GPLv3 terms, with no additional restrictions. Upstream copyright, source availability, and asset notices remain applicable.

The source corresponding to a published binary is available from the same release tag and its linked commit. This repository does not provide proprietary builds or private-source binaries under the Mindustry-UX name.

## 中文说明

本项目是 [Anuken/Mindustry](https://github.com/Anuken/Mindustry) 的**非官方**修改版，与 Anuken 及官方项目无隶属或背书关系。

公开版只包含两项改动：

1. **战役波次倒计时**：载入战役区块时强制打开自动波次计时器，修复部分地图或旧存档 `waveTimer=false` 导致的永久“等待中”。
2. **区块继续 / 新游戏 / 恢复上一局**：已有存档的区块提供独立的“继续”和“新游戏”；开始新游戏前会显式备份上一局，之后可确认恢复。

备份规则：每个区块只保留一份显式备份，再次开始新游戏会覆盖；恢复会覆盖当前区块存档；备份只存在本地，不参与云同步；备份失败会中止新游戏，不会删除当前存档。

除两项公开改动外，不包含实验性开发者界面、鼠标操作改造、无限容量、事件日志/HUD 重做等功能。首次预发布请先备份游戏数据。

Release 中的 Android APK 为 **debug 签名测试包**，包名仍是官方的 `io.anuke.mindustry`，安装前需卸载官方版，不适合分发或长期使用。Windows 包为自带 Java 运行时的 x64 应用目录压缩包，解压后运行 `Mindustry-UX.exe`；该程序未做代码签名，SmartScreen 可能提示风险。

构建需要 JDK 17，桌面产物为 `desktop/build/libs/Mindustry.jar`，服务端产物为 `server/build/libs/server-release.jar`。

本项目遵循 **GPLv3**，保留原始许可证、上游版权与署名，不附加额外限制；每个二进制版本对应的源码可从同一 Release 标签获取。
