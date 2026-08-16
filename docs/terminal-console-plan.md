# Terminal console plan

End in mind: a terminal-mode watch. Verbose Lina logs on the color OLED, date/time/Wi-Fi on a split bottom pane, triple-tap Wi-Fi, gray LCD still a clock when the OLED sleeps.

## Layout (not actual tmux)

400×400 round OLED, bezel margin so glyphs are not clipped. Font large, console/fastboot density (~18–22 px), not Asteroid QML.

- **Top ~70%:** scrolling `journalctl -u watch-voice.service -f`
- **Bottom ~30%, top-aligned in that pane:** `/usr/bin/date` (same clock as the voice script) and a dedicated `WIFI ON` / `WIFI OFF` line from ConnMan

Wi-Fi changes (triple-tap or rate-limit disable) log into the voice journal **and** update the bottom line.

## Drawing path

Preferred: small Qt app, `QT_QPA_PLATFORM=hwcomposer`, no Wayland desktop.

Blocked until LPM/gray LCD is solved without (or with a stub for) Lipstick. See [display-and-lcd.md](display-and-lcd.md).

## Triple-tap

Read `/dev/input/event1` (`fts_ts`) only when MCE display state is **on**. Three taps → `connmanctl enable wifi` or `disable wifi` once. Log `watch-voice: wifi on` / `wifi off`.

## Voice script

Minimal: log ConnMan wifi powered/connected flips. `wait_for_transport` already pauses until wifi is back. No extra sleep after rate-limit clips.

## Order (stop between steps for on-wrist tests)

1. Spike: stop launcher, prove MCE unblank **and** gray LCD. **Unblank works; gray LCD does not. Launcher restored.**
2. Resolve compositor LPM (stub vs keep Lipstick/QML-only).
3. `watch-term` two-pane console as a ceres user service; install via watch-voice `install.sh` once it exists.
4. Triple-tap helper (build armhf; watch has no comfortable toolchain).
5. Persistently disable `asteroid-launcher` only after gray LCD + console are proven. Leave packages installed.
6. CPU `performance` @ 1.094 GHz is a later conversation.

## Watch-voice install reminder

Do not ship GitHub-only runtime. Install to the watch first. Never commit `groq.key`.
