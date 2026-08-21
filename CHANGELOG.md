[CHANGELOG.md](https://github.com/user-attachments/files/31295551/CHANGELOG.md)
# Changelog

## v4.1.16 (Latest)

### Added
- **Automatic background update check** — runs once, ~5s after launch,
  against the same manifest Tools > About > Check for Updates already
  used. Silent when there's nothing new; when there is, it shows a small
  dot on Tools and About, a tray menu item ("⬆️ Update available: vX.Y.Z"),
  and (where supported) a Windows notification — without forcing a modal
  open or interrupting anything.
- **Skip this version** — next to the Download Update button, lets you
  dismiss a specific release so the background check won't re-notify about
  it (it'll still notify about anything newer). Stored per-PC in
  `update-prefs.json` under the app's userData folder.
- **Update checksum verification** — the manifest can now carry an
  optional `"sha256"` field; after a download finishes, the file is hashed
  and compared, and the result ("✅ Checksum verified" / "⚠️ Checksum
  mismatch") is shown next to the download. Manifests without a `sha256`
  field behave exactly as before — no checksum note is shown.
- **Install Now** button after a successful download — launches the
  installer directly and quits the app, instead of only offering "Show in
  Folder" and leaving the rest to you. Hidden if the checksum came back a
  confirmed mismatch.

## v4.1.15

### Fixed
- **Update download had no visible progress.** The "Download" link on
  Tools > About > Check for Updates was a plain `target="_blank"` anchor;
  Electron opened it as a blank, chrome-less popup window and let the file
  save to the Downloads folder silently in the background — nothing in the
  app ever indicated a download was happening, even though it was. Replaced
  it with a "Download Update" button wired through a real `will-download`
  handler in the main process, so the button now shows live percentage
  progress and, once finished, a "Show in folder" action.

## v4.1.14

### Added
- **Restart with new settings** (🔄 button on each session) — stops and
  immediately relaunches a single session using whatever's currently saved
  in Configuration, without touching any other active session. Useful after
  tweaking bitrate/codec/grid settings mid-session, since scrcpy has no way
  to apply new flags to an already-running mirror.
- **Wi-Fi latency graph** — a 📈 button next to each session's link-quality
  badge opens a modal chart of the last 30 readings (about 2 minutes at the
  existing 4s poll rate), so a link degrading gradually is visible before
  it drops to "poor". (Originally shipped as an inline text sparkline that
  widened with every reading until it hit its 30-reading cap, making
  session rows visibly grow; replaced with this on-demand graph so rows
  stay a fixed width.)
- **Copy connection string** (📋) — on session rows and in the device
  picker, copies the raw `ip:port` or USB serial to the clipboard.
- **Mini HUD** (Tools > Mini HUD) — a small, frameless, always-on-top
  window listing active sessions with owner, model, battery, and Wi-Fi
  quality, so you can keep an eye on things without the full app window
  in focus.
- **Panic-button feedback** — `Ctrl+Alt+Q` now shows a toast notification
  confirming how many sessions it stopped, instead of stopping silently.
- **Grid columns** override in Configuration — pick a fixed column count
  for the grid window layout (e.g. always 2-up) instead of always letting
  it auto-compute a square-ish layout from the current session count.
- **Per-device trusted defaults** — "Save as default for this device" in
  the device picker lets a trusted (auto-connect) phone remember its own
  mirror settings, used instead of the global last-used config the next
  time it auto-connects.
- **Recent Captures quick actions** — copy path, push a capture back to
  the active device, and delete, all inline in the Recent Captures list.
- **App Launcher favorites** — pin (⭐) frequently-launched packages to the
  top of the list, and export/import the pinned list as a JSON file to
  carry it to another PC.

### Changed
- **Wi-Fi auto-reconnect** now backs off exponentially (5s, 10s, 20s, 40s,
  60s, capped) instead of always retrying after a fixed 5s, and logs a
  clear "gave up after N attempts" notification once retries are
  exhausted instead of just quietly stopping.

---

## v4.1.13

### Added
- **Global hotkeys** — `Ctrl+Alt+S` screenshots the most recently connected
  session from anywhere (no need for the app or a mirror window to have
  focus); `Ctrl+Alt+Q` stops every active session immediately.
- **Show system apps** toggle in the App Launcher (previously always
  filtered to user-installed apps only).
- **Check for Updates** button in Tools > About. The manifest URL is now
  configurable from Tools > About itself (stored in `update-config.json`
  under the app's userData folder) instead of hardcoded in `main.js`, so it
  can point at a GitHub-hosted `{ "version", "url", "notes" }` JSON file
  without a rebuild.

### Fixed
- `Stop` on a session that had already exited on its own no longer fails
  silently — a genuine stop failure is now logged instead of swallowed.
- Fixed a stray duplicate `</ul>` in the Instructions tab markup.

### Internal
- The most-recently-started session's device id is now tracked, so
  "act on the active device" features have a sensible target when more
  than one phone is mirroring at once.

---

## v4.1.12

### Added
- **Trusted Devices (Auto-Connect)** — mark a phone trusted from the device
  picker or Manage Phones; it then skips the Select Device popup and starts
  mirroring automatically as soon as it's detected. Stored per-PC in
  `trusted-devices.json` under the app's userData folder — separate from
  owner/password registration, which is stored on the phone itself.
- **Recent Captures** browser (Tools menu) — lists every screenshot and
  recording the app has saved (matched by the `AM_` filename prefix),
  newest first, with image thumbnails and a "Show in Explorer" action.
- **Clipboard-only mode** — a new Configuration toggle that connects with
  `--no-video`, giving keyboard/mouse control and clipboard sync without
  opening a mirror window at all.

### Fixed
- Closed a path-traversal gap in the drag-and-drop push destination: a
  destination folder containing `..` segments is now rejected instead of
  silently accepted.

### Internal
- `takeScreenshot` extracted into a reusable helper shared by the IPC
  handler and the new global hotkey.

---

## Known limitations / not included in this release

Flagged for anyone continuing this work — these were suggested but are
either out of scope for a same-session patch, need infrastructure this
repo doesn't have yet, or need real-device testing to land safely:

- **Real auto-updater** (`electron-updater` + signed release feed). The
  Check for Updates button is a lightweight manual check against a static
  JSON manifest, not a full silent-update pipeline.
- **Notification mirroring** (forwarding Android notifications to Windows).
- **Multi-device input broadcast** (same input to several phones at once).
- **Persistent adb shell for Wi-Fi quality polling** (currently spawns a
  short-lived `adb` process per ping tick — works, just not the most
  efficient at high device counts).
- **Full TypeScript conversion / module split** of `main.js` and
  `renderer.js` — both were extended in place rather than restructured, to
  avoid introducing regressions in a single pass without a device to test
  against.
- **`.ps1` subnet-mask generalization** — the discovery script still
  assumes a `/24` LAN for its full-sweep fallback.

## v4.1.11
See the in-app Tools > Help & Guides > Updates tab for the full history
through v4.1.7.
