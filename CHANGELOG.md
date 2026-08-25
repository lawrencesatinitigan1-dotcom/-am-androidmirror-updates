[Uploading CHANGELOG.md…]()
# Changelog

## v4.1.23 (Latest)

### Added
- **Session Configuration is now editable.** Opening ⚙️ on an Active
  Sessions row no longer just shows a read-only snapshot — every setting
  (bitrate, resolution, FPS, codecs, keyboard/mouse mode, orientation
  lock, audio, and all fifteen toggles) is now a live field in the modal,
  using the same option sets as Tools > Configuration.
- **⚡ Apply Now button.** Restarts just that one device with the edited
  settings immediately, via the same `restartMirror` plumbing the existing
  🔄 "Restart with current Configuration settings" button uses — every
  other active session keeps mirroring undisturbed. If Recording gets
  switched on from here, a fresh save-file prompt is resolved first the
  same way starting a brand-new session already does, so the toggle
  doesn't silently do nothing.

## v4.1.22

### Fixed
- **Version badge next to the app name could render clipped.** The
  `.app-version` pill had no `white-space`/`flex-shrink` control inside the
  header's flex layout, so on a narrower window the version string could
  wrap onto a second line or get squeezed enough to lose a trailing
  character. It's now pinned to one line and can't shrink, so the full
  `vX.Y.Z` always renders.

### Added
- **Tooltips on every Session Configuration row.** Opening ⚙️ on an Active
  Sessions row now explains what each setting actually does when you hover
  its name, including exactly what changes if it's **On** vs. **Off** for
  every toggle (Forward Device Audio, Stay Awake, Show Touches, View Only,
  Clipboard Sync, Auto-Reconnect, Recording, Grid Layout, and the rest),
  reusing the same overflow-safe tooltip already used for Active Sessions
  row buttons so nothing gets clipped scrolling through a long list.
- **Mini HUD now follows the app's theme.** Previously the HUD was
  permanently dark no matter what was picked in Tools > Appearance. It
  reads the same saved theme (or "Sync with Windows theme" preference,
  live) as the main window via the shared `localStorage` the two windows
  already use for registered-device data, including custom colors - no new
  main-process plumbing required.

### Changed
- **Magnified shortcuts list.** Rows in ⌨️ Shortcuts now gently zoom in as
  the cursor gets close to them, tapering off across neighboring rows
  (dock-style), making it quicker to spot which row you're currently
  scanning in a long list.

## v4.1.21

### Added
- **Reboot Device** (♻️ on each Active Sessions row) — reboots the phone
  via `adb -s <device> reboot`, gated behind a confirmation dialog since it
  ends the current session as a side effect (the phone drops off adb the
  instant it reboots). A Wi-Fi session with **Auto-reconnect** enabled
  will pick back up on its own once the phone finishes booting, same as
  any other unexpected drop.
- **Screen power toggle** (🌙/🔆 on each Active Sessions row) — turns the
  phone's own display off to save battery on a long session, or back on,
  without disconnecting. Input, clipboard sync, and file transfer keep
  working the whole time; only the phone's screen changes. Implemented by
  reading current wakefulness from `dumpsys power` and only sending the
  `POWER` keyevent (`input keyevent 26`) when that would actually flip it
  the requested direction, so the button can't get out of sync with the
  phone's real state. The state is also remembered on the live session and
  included in `get-active-sessions`, so the Mini HUD shows a matching
  "🌙 Screen off" badge even if it's opened *after* the screen was already
  toggled off, not just from that point forward.
- **Export Terminal Log** — a new "💾 Export" button next to **Clear** in
  the Terminal Output header saves the current log to a plain-text file
  via a native save dialog, useful for attaching to a bug report.
- **Do Not Disturb** (Tools menu) — silences ambient Windows notifications
  (update available, low battery, "gave up reconnecting") without
  touching anything else in the app. Notifications that directly confirm
  something you just did — a hotkey screenshot, the panic-button's stop
  count — still show either way, since those aren't the kind of
  interruption DND is meant to quiet. Persisted per-PC in
  `dnd-prefs.json` under the app's userData folder, same pattern as
  Always on Top.
- **Restart ADB Server** (Tools menu) — a one-click fix for a phone that
  isn't being detected, equivalent to running `adb kill-server` followed
  by `adb start-server` from a terminal. Refuses to run while any session
  is active, so it can't yank the connection out from under a live
  mirror — stop your sessions first if you need it.

## v4.1.20

### Fixed
- **Battery badge and low-battery alert didn't recognize USB charging.**
  Charging state was read only from `dumpsys battery`'s `AC powered:`
  flag. Android reports three independent power sources - `AC powered`,
  `USB powered`, and `Wireless powered` - and a phone charging over the
  same USB cable it's mirroring through (by far the most common case in
  this app) reports `AC powered: false`, so it was being read as not
  charging. That meant the battery badge showed no ⚡ for a USB-charging
  phone, and - more seriously - a phone happily charging over USB could
  still trip the "low battery and not charging" notification. All three
  power sources are now checked (`parseChargingState()` in `main.js`,
  shared by the live battery poll and `get-device-details`), and the
  active source is now reported too, so the battery badge's tooltip can
  say which one it is (e.g. "charging via USB") on Active Sessions rows,
  the "Connected to" bar, and the Mini HUD.
- **Recent Captures row buttons (📋 copy path, 📲 push to device, Show,
  🗑️ delete) were unevenly stretched.** Same root cause as the "Copy All"
  fix in v4.1.18 and the App Launcher pin fix in v4.1.19: these buttons
  only ever carried `.btn-modal-action`, which doesn't override the
  global `button` rule's `flex: 1 1 30%` / `min-width: 130px` meant for
  the big toolbar buttons - so inside their own compact 4-across row they
  stretched unevenly instead of sizing to their icon/label. They now have
  their own `.capture-action-btn` sizing, matching the other small icon
  buttons in the app.

## v4.1.19

### Fixed
- **Mini HUD stayed empty if opened after a phone was already connected.**
  `main.js` only ever broadcast session events to the Mini HUD window while
  it was already open, so a session started before you opened Tools > Mini
  HUD was invisible to it - the HUD just showed "No active sessions"
  forever, even with a phone actively mirroring. It now asks the main
  process for a full snapshot of whatever's already running the moment it
  opens, and hydrates itself from that.
- **"Connected to" bar's battery % went stale.** It only ever showed the
  reading taken once at connect time, baked into plain text, while the
  Active Sessions row next to it kept updating live every ~45s. The bar
  now has its own battery badge - visually identical to the Active
  Sessions one - that stays in sync with the same live poll.
- **Installed Apps pin (⭐) button was oversized and unbalanced.** Same
  root cause as the "Copy All" fix in v4.1.18: it was inheriting the
  global button rule's `min-width: 130px` and flex-grow instead of sizing
  to its icon. It now has its own compact, fixed-width sizing to match the
  other small icon buttons in the app.
- **Tools modals (Manage Phones, App Launcher, Recent Captures, session
  config, etc.) could reopen mid-scroll.** Only the outer modal's scroll
  position was reset when opening it; several modals also have their own
  separately-scrolling inner list, which kept whatever scroll position it
  was left at from the last time that modal was open. Every modal now
  resets to the top on open, outer and inner scroll regions alike.

### Added
- **Manual port entry** next to the port slider in Register Phone (Tools
  > Register Phone) - type an exact port instead of dragging the slider,
  including ports outside the slider's 5556–5655 quick-pick range. Leave
  both blank for USB devices, same as before.
- **Tooltips on every remaining Configuration option** that didn't already
  have one - Video Bitrate, Max Resolution, Max FPS, Video Codec, Keyboard
  Input Mode, and every Session Options checkbox now explain what they do
  and which scrcpy flag they map to, matching the existing tooltips on
  Mouse Mode, Orientation, Audio Codec/Bitrate, and Software Window Size.

## v4.1.18

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
