# Lina console watchface (2026-08-16)

Lipstick / `asteroid-launcher` stays. The two-pane console is a watchface, not a second hwcomposer client.

## Why

Stopping Lipstick unblanks a black OLED and **breaks** gray LCD handover (MCE needs `org.nemomobile.compositor` for `LPM_ON`). `--prepare-timepiece` is not a substitute (see `do-not.md`).

## What shipped

| Piece | Path |
|---|---|
| Watchface | `/usr/share/asteroid-launcher/watchfaces/018-lina-console.qml` |
| Feed | `watch-term-feed.service` → `/var/lib/watch-voice/watch-term-feed.sh` |
| Payload | `/var/lib/watch-voice/console.txt` (`date`, `WIFI ON`/`OFF`, SSID or `OFFLINE`/`SEARCHING`/`NO CONNECTION`, `---`, then faux shell). 24h history in `console.hist`. |
| Previous face | `/var/lib/watch-voice/previous-watchface` |
| Triple-tap Wi-Fi | Watchface taps (OLED on or dimmed), not on the top/bottom 16% edge bands. The evdev helper `watch-triple-tap.service` is leftover and must stay **disabled**. |
| Shell scroll | Vertical phone-style flick inside the top pane only. Does not cover swipe-from-top or swipe-from-bottom. New journal lines jump to the last `$ `. |
| Bottom short-press Wi-Fi | `watch-bottom-wifi.service` reads `gpio-keys` event2 (`KEY_MENU` 139) without grab. Works on the gray LCD. Does not unblank. Ignore holds ≥1s (10s is PMIC reset). |
| Wi-Fi heal after enable | `watch-wifi-heal.service` watches ConnMan wifi `Powered`. If scan says `No carrier` / `wpa_supplicant` stays `inactive`, it restarts `wpa_supplicant` so saved APs can autoconnect. This is the disable-then-enable bug on the wrist, not USB. Do not restart `connman` (that can bounce RNDIS). |
| Archive | Long-press the pink shell, then double-tap the prompt. `watch-voice.sh` moves `hist.txt` to `/var/lib/watch-voice/archive/`. |

Source: `~/.local/share/watch-voice/term/` and `install-term.sh`. Re-running the installer is the permanent path: it copies the face, enables the feed, writes `/desktop/asteroid/watchface`, restarts Lipstick, then `mcetool -E disabled`.

## Layout

Top ~60%: faux `$ ` shell from the voice journal (`$ tinysay "…"`, `$ tinycap`, …). DejaVu Sans Mono at 26 px. Failures are red debug echoes, not pink `$ ` lines. History is kept 24 hours on disk (`console.hist`) and is swipe-scrollable up/down in this pane only. The watchface leaves the top and bottom 16% of the screen to asteroid-launcher (swipe-from-top = Quick Settings, swipe-from-bottom = app launcher). A new echo pins the last `$ ` to the bottom of the top pane. Display-only transform in `watch-term-feed.sh`; the real journal stays `watch-voice:` for debug.

Bottom ~40%, top-aligned:

```
              Monday
2026-08-17                    09:08 WIB
WIFI ON                          BAT 99%
           Cyberdeck2024
```

Weekday centered (Montserrat, with date/time). Date left, time+TZ right. WIFI left, BAT right (Barlow). Bottom line is the SSID when associated (white), `OFFLINE` if Wi-Fi is powered off (blue), `SEARCHING` while looking for a saved AP (pink), `NO CONNECTION` if the radio is up but no recognized AP is in range (red). Battery from `nanohub_fuelgauge-0/capacity` **once per OLED on** (not on the 1 Hz feed).

Palette: hot-pink shell (`#ff4fd8`, DejaVu Sans Mono at 26 px). Errors are red (`#ff3b3b`). Date and WIFI blue. Time, BAT, and weekday purple.

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
