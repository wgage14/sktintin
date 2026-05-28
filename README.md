# sktintin

A [TinTin++](https://tintin.mudhalla.net/) configuration for playing
[Shattered Kingdoms](https://www.shatteredkingdoms.org/) (`mud.shatteredkingdoms.org:1996`)
from Termux on Android.

The repo is edited on a desktop and pulled to the phone — no GUI client
required on mobile.

## What it does today

- **Timestamped connect/disconnect events** — every `SESSION CONNECTED`,
  `SESSION DISCONNECTED`, and `SESSION TIMED OUT` is printed in-session
  with a `YYYY-MM-DD HH:MM:SS` prefix. See [events.tin](events.tin).
- **8-direction tap compass** pinned to the upper-right corner, sized
  for portrait play. Taps map to `n / ne / e / se / s / sw / w / nw`.
  Button size and spacing are tunable (see below).
- **Tick timer** along the bottom status row: `Next tick in ~37 sec`
  with a block-character progress bar that fills as the tick approaches.
  Increments once per second; ~70 trigger patterns re-anchor it to 0 when
  a tick is observed (regen, hunger, weather, etc.). Weather messages only
  re-anchor near tick boundaries (within 5s) since they're unreliable
  mid-tick. Logic ported from the Mudlet `skmudlet` profile's `ticktimer`
  trigger group.
- **Scrollback via swipe / keyboard** — because the split takes over the
  screen, native terminal scroll is disabled and re-bound to `#buffer`
  (see Scrolling below).

Both UI elements live in [overlay.tin](overlay.tin).

## Files

| File | Purpose |
|---|---|
| [sk.tin](sk.tin) | Entry point. Sets config, loads modules, opens the session. |
| [events.tin](events.tin) | Connect / disconnect / timeout timestamping. |
| [overlay.tin](overlay.tin) | Owns the screen split. Top-right compass + bottom-row tick timer + scrollback bindings (`#draw` + `#button` + `#ticker` + `#action`). |

## Screen layout

```
 +----+ +----+ +----+
 | NW | |  N | | NE |   <- compass: 3x3 grid, top-right
 +----+ +----+ +----+      (rows reserved by #split in overlay.tin)
 | W  | |  + | |  E |
 +----+ +----+ +----+
 | SW | |  S | | SE |
 +----+ +----+ +----+
 ------[ scrollback ]------
 Next tick in ~37 sec  ########____   <- row -2 (status)
 >                                     <- row -1 (input)
```

## Tuning button size

Edit the geometry vars at the top of [overlay.tin](overlay.tin):

```
#variable {btn_w} {6}   ;; button width  (columns)
#variable {btn_h} {2}   ;; button height (rows)
#variable {gap_x} {2}   ;; horizontal gap between buttons
#variable {gap_y} {1}   ;; vertical gap between buttons
```

The top split height is derived automatically (`btn_h * 3 + gap_y * 2`),
so bigger buttons reserve more rows up top. Reload with `#read sk.tin`
(or restart) after changing these.

## Scrolling

The `#split` puts TinTin++ in full-screen mode, which disables the
terminal's own scrollback. Scrolling is re-bound in
[overlay.tin](overlay.tin):

- **Finger swipe (phone):** swipe down = scroll up into history, swipe up
  = back toward live output. This relies on TinTin++ `SWIPED` events; the
  direction string can vary by build. If swiping does nothing, run
  `#config {mouse info} {on}`, swipe once, and note the value after
  `SWIPED` — then adjust the `#event {SWIPED}` block to match.
- **Keyboard:** PageUp / PageDown scroll, End jumps back to live output.
- Any `#buffer up` leaves you in scroll-lock; `#buffer end` (or End)
  returns to the live feed.

## Setup on Termux (Pixel 8 Pro)

```bash
pkg update && pkg install tintin++ git
git clone https://github.com/wgage14/sktintin.git
cd sktintin
tt++ sk.tin
```

To pull updates from the desktop side later:

```bash
cd ~/sktintin && git pull && tt++ sk.tin
```

### Termux touch input

The compass uses TinTin++'s `#button` regions, which fire on left-click
events delivered by the terminal's xterm mouse protocol. Termux's
default terminal forwards single-finger taps as left-clicks when mouse
tracking is on (we enable it in [sk.tin](sk.tin) via
`#config {MOUSE TRACKING} {ON}`).

If taps aren't registering:
- Long-press the Termux terminal area → **More** → confirm "Mouse mode"
  isn't intercepting events for selection.
- Some Termux builds need `stty -echoctl` or an explicit terminal
  resize before mouse events flow — switching font size once usually
  triggers a `SCREEN RESIZE` and rebinds the compass.

## Workflow (desktop ⇄ phone)

```
┌─────────────────────────┐         ┌──────────────────────────┐
│  Desktop (this machine) │         │  Mobile (Termux)         │
│                         │         │                          │
│  edit *.tin             │         │  cd sktintin             │
│  git commit / git push  │ ──────▶ │  git pull                │
│                         │         │  tt++ sk.tin             │
└─────────────────────────┘         └──────────────────────────┘
```

## Customizing locally without committing

Anything matching `local.tin` or `*.local.tin` is gitignored. Drop
phone-only tweaks (different keybinds, alternate prompt color,
credentials) into `local.tin` and source it from `sk.tin` with
`#read {local.tin}`.

## Reference

- [TinTin++ manual](https://tintin.mudhalla.net/manual/)
- [`#button`](https://tintin.mudhalla.net/manual/button.php) /
  [`#draw`](https://tintin.mudhalla.net/manual/draw.php) /
  [`#split`](https://tintin.mudhalla.net/manual/split.php) /
  [`#event`](https://tintin.mudhalla.net/manual/event.php) /
  [`#format`](https://tintin.mudhalla.net/manual/format.php)
