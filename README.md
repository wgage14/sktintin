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
- **Tick timer** along the bottom status row showing seconds-until-next-tick
  with a progress bar. Increments once per second; ~70 trigger patterns
  re-anchor it to 0 when a tick is observed (regen, hunger, weather, etc.).
  Weather messages only re-anchor near tick boundaries (within 5s) since
  they're unreliable mid-tick. Logic ported from the Mudlet `skmudlet`
  profile's `ticktimer` trigger group.

Both UI elements live in [overlay.tin](overlay.tin).

## Files

| File | Purpose |
|---|---|
| [sk.tin](sk.tin) | Entry point. Sets config, reserves screen rows via `#split 3 1`, loads modules, opens the session. |
| [events.tin](events.tin) | Connect / disconnect / timeout timestamping. |
| [overlay.tin](overlay.tin) | Top-right compass + bottom-row tick timer (`#draw` + `#button` + `#ticker` + `#action` reset patterns). |

## Screen layout

```
 NW   N  NE     <- rows 1-3 reserved by #split (compass)
  W  ( )  E
 SW   S  SE
------------
[ scrollback ]
------------
Tick ~37s [###....]   <- row -2 (bottom status, reserved by #split)
>                     <- row -1 (input bar)
```

## Setup on Termux (Pixel 8 Pro)

```bash
pkg update && pkg install tintin++ git
git clone https://github.com/wgage14/sktintin.git
cd sktintin/sktintin
tt++ sk.tin
```

To pull updates from the desktop side later:

```bash
cd ~/sktintin/sktintin && git pull && tt++ sk.tin
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
│  Desktop (this machine) │         │  Mobile (Termux)    │
│                         │         │                          │
│  edit *.tin             │         │  cd sktintin/sktintin    │
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
