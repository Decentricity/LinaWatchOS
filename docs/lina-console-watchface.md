# Lina console watchface (2026-08-16)

Lipstick / `asteroid-launcher` stays. The two-pane console is a watchface, not a second hwcomposer client.

## Why

Stopping Lipstick unblanks a black OLED and **breaks** gray LCD handover (MCE needs `org.nemomobile.compositor` for `LPM_ON`). `--prepare-timepiece` is not a substitute (see `do-not.md`).

## What shipped

| Piece | Path |
|---|---|
| Watchface | `/usr/share/asteroid-launcher/watchfaces/018-lina-console.qml` |
| Feed | `watch-term-feed.service` → `/var/lib/watch-voice/watch-term-feed.sh` |
| Payload | `/var/lib/watch-voice/console.txt` (`date`, `WIFI ON`/`OFF`, `---`, then faux shell) |
| Previous face | `/var/lib/watch-voice/previous-watchface` |
| Triple-tap Wi-Fi | Watchface `MouseArea` (OLED on or dimmed). The evdev helper `watch-triple-tap.service` is leftover and must stay **disabled**. |
| Bottom short-press Wi-Fi | `watch-bottom-wifi.service` reads `gpio-keys` event2 (`KEY_MENU` 139) without grab. Works on the gray LCD. Does not unblank. Ignore holds ≥1s (10s is PMIC reset). |
| Archive | Long-press the pink shell, then double-tap the prompt. `watch-voice.sh` moves `hist.txt` to `/var/lib/watch-voice/archive/`. |

Source: `~/.local/share/watch-voice/term/` and `install-term.sh`. Re-running the installer is the permanent path: it copies the face, enables the feed, writes `/desktop/asteroid/watchface`, restarts Lipstick, then `mcetool -E disabled`.

## Layout

Top ~70%: faux `$ ` shell from the voice journal (`$ tinysay "…"`, `$ tinycap`, …). Display-only transform in `watch-term-feed.sh`; the real journal stays `watch-voice:` for debug.

Bottom ~30%, top-aligned:

```
2026-08-16                    18:48 WIB
WIFI OFF                         BAT 85%
              Sunday
```

Date left, time+TZ right. WIFI left, BAT right. English weekday centered under that. Battery from `nanohub_fuelgauge-0/capacity` **once per OLED on** (not on the 1 Hz feed).

Palette: hot-pink shell (`#ff4fd8`). Bottom left (date, WIFI) blue; WIFI OFF is dimmer blue. Bottom right (time, BAT) and weekday purple.

After `asteroid-launcher` restart, **do not** force `mcetool -E enabled`. On catfish that is OLED doze (AOD), which blocks the gray FSTN. Match `/org/asteroidos/settings/always-on-display` (false here). USB nightstand AOD is also off (`/desktop/asteroid/nightstand/always-on-display`).

The console watchface must not redraw unless MCE `DisplayOn`. A 1 Hz refresh after blank keeps the AMOLED up, so neither the top key nor idle timeout can hand off to the gray LCD.

**Confirmed 2026-08-16:** top button and idle timeout both switch to the gray digital LCD. Color console, split clock, centered weekday, and palette look right. Triple-tap Wi-Fi on the color console works.

History is not wiped on Wi-Fi off, idle, or old turns. Archive is the shell long-press. Wi-Fi Powered off/on play `wipe.wav` / `wifi-on.wav`.

## Rollback (watchface only)

```
su -s /bin/sh ceres -c 'env XDG_RUNTIME_DIR=/run/user/1000 DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus dconf write /desktop/asteroid/watchface '"$(cat /var/lib/watch-voice/previous-watchface)"
systemctl --user -M ceres@ restart asteroid-launcher.service
mcetool -E disabled
mcetool --unblank-screen
```

Do not stop Lipstick to “go back.” Analog shades is enough.

## Do not in this step

- Stop or disable `asteroid-launcher`
- Enable `watch-triple-tap.service` (Lipstick already owns `fts_ts`)
- `lcd-tools --prepare-timepiece`
- Write `/dev/fb0` or force `lcd-backlight` to 0
