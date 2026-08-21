[CHANGELOG.md](https://github.com/user-attachments/files/31324388/CHANGELOG.md)
# Changelog

## v4.1.18 (Latest)

### Fixed
- **"Copy All" button was oversized.** A global button rule (min-width
  130px, flex-grow) intended for the big toolbar buttons was also
  stretching this small Active Sessions header button. It now has its own
  compact sizing.

### Changed
- **Main window opens a little larger by default** — 700×740, up from
  640×700 — and now **remembers its size and position** per-PC across
  restarts (saved to `window-prefs.json` under the app's userData
  folder) instead of always resetting to the same spot. A saved position
  that no longer lands on any connected display is ignored so the window
  can't open off-screen.
- **"Copy All" now includes owner names** — each line is now
  `Owner — connection string` instead of just the bare connection
  string.

### Added
- **Always on Top** (Tools menu) — keep the main app window above other
  windows, independent of the Mini HUD (which is always-on-top by
  design). Remembered per-PC in the same `window-prefs.json`.
- **Live session count in the tray icon's tooltip** — hovering the tray
  icon now shows how many sessions are active without opening the
  right-click menu.
- **Software Window Size** dropdown (Configuration) — pick how the main
  app window opens: **Remember Last Size** (default, same behavior as
  before), **Compact, Default, Large**, or the new **Full Maximize**.
  Applies immediately on Save and is remembered per-PC for the next
  launch (stored in `window-prefs.json` alongside the existing
  remembered-geometry data). This controls the main AM window, not a
  mirror window.
- **Auto-show main window on mirror close** — if any mirror session
  closes (stopped, disconnected, or failed to connect), the main app
  window automatically comes back to the front, so it doesn't stay
  hidden behind a mirror window after that mirror is gone. Applies to any
  single session closing, not just the last one.
- **⚙️ Show configuration** button on each Active Sessions row — opens a
  read-only summary of the exact settings that session is currently
  running with, read straight from its live process state in the main
  process (not from whatever's currently sitting in the Configuration
  modal, which may have changed since that device connected).

## v4.1.17

### Changed
- **Check for Updates is now the Download button.** Finding an update used
  to add a separate "Download Update" button below "Check for Updates",
  stacking two buttons in a small modal. The same button now handles both
  jobs: it reads "Check for Updates" until it finds something, then
  becomes "Download Update (vX.Y.Z)", then "Show in Folder" once the
  download finishes — all in place, no second button.
- **Background update check now repeats every 6 hours** instead of only
  running once ~5s after launch, so a release that ships while the app is
  left open mid-session still gets caught (still respects "Skip this
  version").

### Fixed
- **Mini HUD never actually received live session updates.** It listened
  for the same `onScriptStarted` / `onMirrorStopped` / `onMirrorQuality`
  IPC events as the main window, but `main.js`'s event broadcaster only
  ever sent them to the main window - so the HUD stayed on "No active
  sessions" no matter what was running. It now gets everything the main
  window gets.

### Added
- **Live battery tracking** — every active session (USB or Wi-Fi) now
  polls battery % and charging state roughly every 45 seconds for the
  life of the session, instead of only reading it once at connect time.
  Shown as a badge on each Active Sessions row and kept current in the
  Mini HUD.
- **Low-battery notification** — a Windows notification fires the first
  time a mirrored device drops to 15% or lower while unplugged. Re-arms
  once the device recovers above 20% or starts charging, so it won't
  repeat every poll while sitting right at the threshold.
- **Session uptime badge** on every Active Sessions row and Mini HUD
  entry, ticking live.
- **Copy All** button on the Active Sessions panel — copies every active
  session's connection string as a newline-separated list in one click.

## v4.1.16

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
