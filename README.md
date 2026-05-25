# Window Gestures (GNOME 49 Fix)

Fork of [amarullz/windowgestures](https://github.com/amarullz/windowgestures) with fixes for GNOME Shell 49 compatibility.

## GNOME 49 Changes

The upstream repo added GNOME 49 to `metadata.json` ([PR #54](https://github.com/amarullz/windowgestures/pull/54)) but did not update the code for GNOME 49's breaking API changes, causing the extension to crash on load (see [issue #55](https://github.com/amarullz/windowgestures/issues/55), [#58](https://github.com/amarullz/windowgestures/issues/58), [#59](https://github.com/amarullz/windowgestures/issues/59)).

### Fixes applied

- **`Meta.Rectangle` removed in GNOME 49** - Replaced with `Mtk.Rectangle` (available since GNOME 45).
- **`window.get_maximized()` removed** - Replaced with `window.is_maximized()` on GNOME 49+. This was the main crash (`TypeError: activeWin.get_maximized is not a function`).
- **`window.maximize(flags)` / `unmaximize(flags)` no longer accept arguments** - Calls are now argument-free on GNOME 49+.
- **Gesture signal/actor change (GNOME 49.4+)** - `TouchpadSwipeGesture` moved from `global.stage` with `captured-event::touchpad` to a per-gesture actor with `event::touchpad`. Handled via feature detection (`g._actor`) so it works on both 49.0-49.2 and 49.4+.

All changes are backward-compatible with GNOME 45-48.

### Known issue: horizontal swipe crash (GNOME Shell bug)

There is a separate [GNOME Shell bug](https://gitlab.gnome.org/GNOME/gnome-shell/-/issues/8850) where horizontal 3/4-finger swipes crash with `this._pages[pageIndex] is undefined`. This is a bug in GNOME Shell itself (not the extension) and is fixed in GNOME Shell 49.4+. Ubuntu 25.10 ships gnome-shell 49.0-49.2 which does not include this fix.

### Build & install

```bash
bash build.sh
gnome-extensions install --force windowgestures@extension.amarullz.com.zip
glib-compile-schemas ~/.local/share/gnome-shell/extensions/windowgestures@extension.amarullz.com/schemas/
```

Log out and back in (Wayland requires a session restart), then enable:

```bash
gnome-extensions enable windowgestures@extension.amarullz.com
```

## Additional fixes (GNOME 49/50 hardening)

Further fixes applied on top of the GNOME 49 compatibility work above.

### Fixes applied

- **Window-close gesture shader crashed on GNOME 49+** - `_createShader()` used the legacy Clutter GLSL API (`Clutter.ShaderEffect` with `Clutter.ShaderType.FRAGMENT_SHADER` / `set_shader_source()`), which Mutter removed. It threw on every close-swipe, flooding the journal with `TypeError: ... ShaderType is undefined` and wasting CPU. The red-tint / dim feedback is now reimplemented on top of `Shell.GLSLEffect` (the supported effect API on GNOME 45-50+) and wrapped so it degrades to "no tint" instead of throwing, should the effect API ever change again.
- **Kinetic fling could peg a CPU core** - `_velocityFlingHandler()` referenced an undefined `target` variable when two flings overlapped, throwing *before* the queue could drain and leaving the 4 ms (~250 Hz) `setInterval` spinning and erroring indefinitely. Fixed the reference (`now.target`).
- **Actor/texture leak on disable** - `destroy()` now releases any leftover on-screen indicator widgets and the cached window-switcher actor, so nothing is orphaned across disable -> enable cycles (screen lock, extension updates). Live window actors (`Meta.WindowActor`) are deliberately left untouched.
- **Defensive gesture-tracker hooks** - `_initFingerCountFlip()` now resolves GNOME's private swipe-tracker internals with optional chaining and drops any that are missing, so a future Shell rename degrades gracefully instead of failing `enable()` (and disabling the whole extension).

### GNOME 50 readiness

- Added `"50"` to `metadata.json` `shell-version`.
- Verified against the [GNOME 49](https://gjs.guide/extensions/upgrading/gnome-shell-49.html) and [GNOME 50](https://gjs.guide/extensions/upgrading/gnome-shell-50.html) porting guides: the existing `IS_GNOME_49` handling is gated on `GNOME_VER >= 49`, so it already covers GNOME 50, and GNOME 50's documented breaking changes (X11 support removed, `easeAsync()` added, `keyboardManager` / `restart`-signal removals) do not affect this extension.

These changes remain backward-compatible with GNOME 45-48.

---

## Original README:

---

# Window Gestures

Window Gestures is GNOME Shell extension for managing window with touchpad gestures.
Support only for GNOME 45.

## Installation
[![Get from GNOME Extension](./gext.svg)](https://extensions.gnome.org/extension/6343/window-gestures/)


## Features
 * Customizable actions for gestures
 * Support `3` or `4` fingers swipe, pinch & hold
 * Swappable 3 / 4 fingers
 * Enable/disable functions in settings
 * With kinetic gesture
 * With adaptive transition interpolable with gesture state

## Window Gestures
 * Swipe `4` Fingers `up` - **Maximize, fullscreen, restore**
 * Swipe `4` Fingers `up + left` or `right` - **Snap window left/right**
 * Swipe `4` Fingers Down - **Move window** `Tap & Hold Disable`
 * Tap & Hold `4` Fingers - **Move/resize window**

## Configurable Gestures
 * Swipe 4 Fingers Left, Right, Down
 * Swipe 3 Fingers Down, Down+Left, Down+Right, Down+Up
 * Pinch In/Out 3/4 Fingers
 * Tap &amp; Hold to move/resize window

## Actions
 * Minimize window
 * Close window
 * Show desktop
 * Next window
 * Previous window
 * Send window left
 * Send window right
 * Back
 * Forward
 * Brightness up
 * Brightness down
 * Volume up
 * Volume down
 * Mute
 * Media play
 * Media next
 * Media previous
 * Alt+Tab switch
 * Overview
 * Application Grid
 * Quick settings
 * Notification
 * Run (Alt+F2)


## Demo
 [![Improve Touchpad GNOME Experience with Window Gesture](https://img.youtube.com/vi/HHDjraAE6sc/0.jpg)](https://www.youtube.com/watch?v=HHDjraAE6sc)

 [![Gnome Extension - My Window Gestures](https://img.youtube.com/vi/yMUYB3OFpBQ/0.jpg)](https://www.youtube.com/watch?v=yMUYB3OFpBQ)

## License

My Window Gestures are distributed under the terms of the GNU General
Public License, version 2 or later. See the [license](COPYING) for details.
Individual extensions may be licensed under different terms, see each source
file for details.
