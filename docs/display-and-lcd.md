# Display, MCE, and the gray LCD

## Two panels

1. **Color AMOLED** — Linux framebuffer / hwcomposer. MCE owns blank/unblank/brightness.
2. **Gray FSTN LCD** — nanohub segments. Not pixels. Timekeeping when the OLED is off (and even when Linux is off, if nanohub is in timepiece mode).

Handover is: OLED on → color UI; OLED blank → nanohub LCD should show time. That handover is **not** “just turn the backlight off.”

## MCE

`mce.service` + `dsme.service`. CLI: `mcetool`.

Useful:

- `--unblank-screen` / `--blank-screen` / `--blank-screen-lpm`
- `-E, --set-low-power-mode=enabled|disabled`
- Double-tap wakeup policy is `proximity`
- Dim 30 s, blank 3 s (defaults observed 2026-08-16)

**Top short-press** is the daily wake. MCE logs it as `powerkey`; that is still the top key.

ALS brightness is on. Blanking is what should hand the color panel back to the gray LCD.

## What Lipstick is for

`asteroid-launcher` **is** Lipstick: Wayland compositor **and** the QML desktop. It owns D-Bus name `org.nemomobile.compositor`.

MCE will not enter **LPM_ON** (ambient / low-power display, i.e. the gray clock) unless that compositor acks `setAmbientEnabled` / LPM. Without it, MCE logs:

```
no compositor; going to logical off
org.freedesktop.DBus.Error.ServiceUnknown: The name org.nemomobile.compositor was not provided by any .service files
```

OLED unblank **does** work without Lipstick (`Display state: on`, backlight ~76). The color panel is then black because nothing is drawing. The gray LCD does **not** light the segments, because MCE fell through to logical off instead of LPM_ON.

## lcd-tools

`/usr/bin/lcd-tools` (hybris + `libmcutool.so`).

| Flag | Catfish meaning |
|---|---|
| `--sync-time` | `nativeSyncTime` + 12/24h property |
| `--prepare-timepiece` | `autoLowPowerScreen` true, then false, `cutOffScreen`, `wipeBandModeData`, `bandMode` |
| `--session-restart` | Present so the systemd unit does not fail; **no-op on catfish** |
| `--enable-stepcounter` etc. | Nanohub features |

User units (ceres and root): `lcd-sync-time.timer`, `lcd-session-restart.service`.

`--prepare-timepiece` without Lipstick returned `autoLowPowerScreen: 4`, `bandMode: 4`. Do not treat it as a working replacement for compositor LPM until that is understood.

`--blank-screen-lpm` while already off is **denied**. Path is ON → LPM_ON, not OFF → LPM_ON.

Do not leave `mcetool -E` / “Use low power mode” **disabled** if powerkey blanking mode is still `lpm`. That combination unblanks for a blink then drops to OFF while the gray LCD stays put, so the buttons look dead.

## Kernel / sysfs

- `/sys/class/leds/lcd-backlight` — OLED backlight, 0–255. MCE drives this.
- `/sys/devices/virtual/graphics/fb0/msm_fb_lcd_loadswitch` — exists; do not poke blindly.
- `/sys/class/nanohub/nanohub/lcd_mutex` — android-init writes `1` at boot. Do not fight it.
- `android-init` `init.rc` chowns loadswitch/lcd_mutex to uid 1000.

## Kernel console

`vtcon0` is a dummy. There is no fbcon on this panel. A “fastboot-sized” terminal must be a userspace client (Qt hwcomposer or similar), not `getty` on tty1.

## Implication for LinaWatchOS

Stopping Lipstick is fine for a black OLED and for Lina (ALSA does not need it). It is **not** fine for the gray clock until one of these exists:

1. A small process that owns `org.nemomobile.compositor` and acks MCE LPM (and then talks to libmcutool / lcd-tools as Lipstick would), or
2. Lipstick stays running with its QML desktop replaced by the two-pane console (heavier; 108 MB stays).

Do not invent a third path that writes `fb0` or nanohub `lock`.
