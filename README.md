# CrabeLoader Manager v0.4.1 (Windows GUI)

Independent companion to the official [CrabeLoader](https://github.com/LucasLhomme/CrabeLoader) for Disney Infinity 3.0 Gold Edition. **Independent; not officially affiliated with the original projects.**

## Run

Double-click `CrabeLoaderManager.exe`. The EXE has built and Windows-tested.

## Features

- Steam library / common-folder auto detection, manual selection, saved game path.
- Dashboard checks required files and launches the game.
- Official loader installation: fetch latest GitHub release installer, verify release SHA-256 when provided, run official installer. The official installer protects foreign DLLs and performs rollback.
- Mods: discover standalone Lua and packaged mods, show manifest version/authors/description, install one-mod ZIP or Lua, enable/disable through active Crabe profile, move removed mods into local recovery backup.
- Profiles: list, create (copy active enabled set), activate. Other TOML sections are preserved; backups created before changes.
- Diagnostics: last 120 lines of `loader.log`, malformed mod entry warnings, duplicate mod IDs, and **heuristic** same-filename collisions in `characters/` and `skilltrees/`.
- Optional ZIP/Lua drag-and-drop with `tkinterdnd2`.

## Limits and safety

- Conflict detection is a heuristic: different filenames may still patch the same game resources; same filenames may be harmless. No automatic conflict resolution.
- ZIP import accepts a single package with `mod.json` or a single `.lua`, not multi-mod collections. Untrusted mods may execute arbitrary code when loaded by the game; inspect sources.
- The app does not automatically update third-party mods. Loader updates are delegated to the official installer.
- Native `.asi` plugins need a full game restart; Lua mods can use F4 hot reload.
- `dinput8.dll` may belong to another ASI loader: presence does not establish that it is CrabeLoader's bootstrap. Official installer handles this.
- Avoid changing mods while the game is running. Do not use on your only copy of important save data without backups.
- Profile editing supports normal single-line TOML `enabled` arrays. Multiline arrays and advanced TOML constructs may require manual editing.
- Config and removed-mod recovery directory: `%LOCALAPPDATA%\\CrabeLoaderManager`.

## v0.3: Build stamp and assets integrity

The **Integrity** tab reads `buildstamp.lua` beside `DisneyInfinity3.exe` as plain text (does not execute Lua), displays the source branch, Git revision and Perforce revision, and checks the game folder's `assets/` (or `Assets/`) directory. It computes a streaming MD5 for each file without modifying it. The first **Create Baseline** action records hashes as a JSON manifest under `%LOCALAPPDATA%/CrabeLoaderManager/baselines/`; **Verify Against Baseline** reports matching, changed, missing and added files. Import/Export let you compare with a known-good manifest from the *same build*. A baseline made from an already damaged installation is not proof of clean game assets. MD5 is for accidental corruption/change detection, **not security authenticity** (use trusted SHA-256 hashes for that). Scans run on a worker thread and support cancellation. Files added/removed while scanning can produce errors or misleading results; close the game before scanning. The app currently expects `assets` directly under the game directory; other layouts are not yet supported. No asset files are deleted or repaired.


## v0.3.1 integrity scan update
- Choose a baseline JSON independently of the game folder. Existing v0.3 JSON manifests remain compatible.
- Choose any target `assets` directory, including a different game installation. Paths are compared relative to the selected assets directory.
- Byte-level scan progress, current filename, MiB/s, and cancellation while enumerating and hashing.
- GUI progress is delivered via a queue rather than Tkinter calls from worker threads.
- Select **Integrity > Browse...** next to Baseline JSON, then **Browse...** next to Target assets folder, then **Verify Against Baseline**.
- For baseline creation, select target folder then **Create Baseline**; it writes to the selected baseline path or the default game baseline path.
- Scanning remains read-only for game assets; creating a baseline only writes the JSON manifest.

### v0.3.2: Two-phase integrity verification
Verification uses the baseline file list directly (no initial target enumeration).
It verifies sizes and MD5 hashes first, then automatically recursively scans the
selected target directory for extra files. The second phase shows an indeterminate
progress bar and live counts. Baseline creation still enumerates all files.
Verification remains read-only; no game files are modified.

## v0.3.3: Loader backup / uninstall / restore
- **Backup Loader** creates a timestamp-independent unique recovery directory under `%LOCALAPPDATA%/CrabeLoaderManager/loader_backups/`, with SHA-256 hashes and copies of present `CrabeLoader.asi`, `dinput8.dll`, `Crabe/crabe.toml`, and `loader.log`.
- **Uninstall Loader** first creates that backup, then removes **only `CrabeLoader.asi`**. This safely disables CrabeLoader without assuming `dinput8.dll` belongs to it. Game assets, mods, configuration, and logs remain untouched. This is a *safe disable/uninstall of the CrabeLoader component*, not a complete removal of every loader-related file.
- **Restore Backup** selects a backup folder and restores missing files, verifying SHA-256. It preserves existing config/log files, refuses to overwrite an existing `CrabeLoader.asi` or different `dinput8.dll`, and checks that the backup belongs to the selected game path.
- All operations require the game to be closed. **The loader backup does not include installed mods or game assets.**

## v0.3.5: CrabeMenu integration

This update adds the official [CrabeMenu](https://github.com/LucasLhomme/CrabeMenu) as a companion mod to the CrabeLoader installer. The **Dashboard** has a default-enabled checkbox **Install / update CrabeMenu after a successful loader installation**. After the official CrabeLoader installer exits with success and `CrabeLoader.asi` is detected, the manager downloads CrabeMenu's latest stable release ZIP from GitHub, verifies the release SHA-256 digest (when provided by GitHub), validates the mod manifest, then installs to `<game>/mods/crabemenu/`. The manager does **not** run the repository's developer-specific `deploy.ps1` script.

The Dashboard also provides **Install / Update CrabeMenu** and **Remove CrabeMenu (backup)** buttons, and shows the installed mod version. Updates create a timestamped recovery copy under `%LOCALAPPDATA%/CrabeLoaderManager/RemovedMods/`; removal moves only `mods/crabemenu` into recovery storage. Existing unrelated mods, CrabeLoader files, and assets are untouched. Existing installations without CrabeLoader are prompted to install the loader first. CrabeMenu requires **CrabeLoader 1.1.0 or newer**; the manager currently checks for `CrabeLoader.asi` but does **not** reliably determine its binary version. Use the latest official CrabeLoader installer.

For testing, run `py -3 -m unittest discover -v`. To build the Windows executable, run `build_exe.bat`; it stores the entire PyInstaller log in `build_error.txt` and shows the last forty lines on failure. The EXE itself cannot be built/tested in this Linux environment, and the official GitHub ZIP is downloaded **on the user's machine at install time**, not bundled into this project ZIP.

## v0.3.5 — CrabeMenu deployment diagnostics

The Dashboard's **Check Deployment / Fixes** button audits the installation
for missing `CrabeLoader.asi` / `dinput8.dll`, incomplete `mods/crabemenu`
packages, UTF-8 BOM in menu Lua files, old `mods/crabemenu.lua` and
`mods/playable_anywhere`, legacy `bink2w32.dll` proxies, and possible obsolete
root-level `api/`, `characters/`, `skilltrees/` folders.

With confirmation and the game closed, the safe fix action backs up and strips
BOM from **CrabeMenu Lua files only**, and moves legacy mods into a timestamped
backup at `%LOCALAPPDATA%\\CrabeLoaderManager\\deployment_backups`.
It does **not** automatically delete obsolete folders, change `bink2w32.dll`,
copy local development DLLs, or touch unrelated mods. These require explicit
manual review or use of the official CrabeLoader installer. The supplied
upstream `deploy.ps1 -Full` contains developer-specific hardcoded paths and
broad deletion actions, so it is deliberately not executed.

The Diagnostics tab includes the deployment findings. Run `python -m unittest discover -v`
for tests. This version has not been validated on a live Windows game installation.

## v0.3.6 — Correct fresh CrabeMenu validation

- `skilltrees/` and `SkillPointsEarned.patch` are generated on demand; neither is required by the deployment checker or ZIP installer.
- The checker requires `mod.json`, `main.lua`, and the `core/`, `modules/`, and `ui/` directories.
- The release ZIP installer validates these components and preserves all files beneath the package's mod root, including new folders introduced by future releases.
- The checker still reports missing loader components independently and leaves game files untouched unless a repair is explicitly approved.

## v0.3.7 — CrabeMenu folder-path diagnostics
- Detects CrabeMenu by `mod.json` even when installed under a versioned folder such as `mods/CrabeMenu-v1.1.0/`.
- Reports literal `mods/crabemenu/` references with script filename and line number when the menu is misplaced. This is a heuristic, not a full Lua interpreter.
- Offers a confirmed, backed-up rename to `mods/crabemenu/` only when exactly one misplaced menu exists and the target is free. Never merges or overwrites an existing directory.
- Blocks installing another copy of CrabeMenu while a misplaced copy exists.
- Only offers BOM/legacy-mod fixes when actionable findings exist.


## v0.3.8
- Compact CrabeMenu path mismatch dialog with **View Details**, **Repair**, and **Cancel**. The detailed findings are shown in a separate scrollable window.
- When normal game launch fails specifically with Windows access-denied (5) or elevation-required (740), ask before retrying with UAC (`ShellExecuteW` / `runas`). Other launch failures are reported normally; no automatic elevation.
- Repair continues to back up the misplaced menu before renaming and refuses to overwrite an existing canonical folder.

## v0.3.9 — Configuration and loader version

- Dashboard displays the installed CrabeLoader version **when the installed CrabeLoader.asi contains a Windows VERSIONINFO resource**. Otherwise it explicitly reports "Unknown" rather than assuming the latest release is installed.
- New **Configuration** tab edits `<GameRoot>/Crabe/crabe.toml`, including general, display, keybinds, updates, plugins, and raw TOML for profiles or advanced settings.
- **Save Form** preserves unrelated settings and comments in the loaded TOML; **Save Raw TOML** validates and saves the editor contents. Both make a `.manager-bak` backup of an existing file before replacement.
- If no configuration exists, the editor offers the documented defaults and saves only on request. Requires Python 3.11+ for `tomllib`.
- Note: the form and raw editor are independent; choose the corresponding Save button. Version detection is not guaranteed for releases lacking embedded file-version metadata.

## v0.4.0 — Lua Security Scanner
- New **Security** tab: **Scan Installed Lua Mods** recursively audits `.lua` files in `mods/`; **Scan Mod ZIP** audits Lua inside a ZIP without extracting or executing it.
- Reports suspicious shell execution, native library loading, dynamic execution, network APIs, file deletion, credential/startup paths, encoded payload indicators, and ordinary file access with severity, source filename, line, and excerpt.
- Read-only; does not disable or modify mods. Large files are skipped and reported; archives have size/file limits.
- Heuristic matching can flag harmless code and miss obfuscated or indirect malicious behavior. It does not inspect `.asi`/`.dll` binaries, and a clean result is **not a safety guarantee**. The scanner currently ignores single-line comments but does not fully parse Lua syntax.


## v0.4.1 — About & Credits

The GUI includes an **About** tab with clear project attribution:

- **CrabeLoader** — created by **Crabe** (original mod loader).
- **CrabeMenu** — created by **Crabe** (original in-game menu).
- **CrabeLoader Manager** — created and maintained by **Weresthebeef** (independent manager).

This manager is an independent companion utility and does not claim authorship of CrabeLoader or CrabeMenu. The About tab links to the original project repositories.
