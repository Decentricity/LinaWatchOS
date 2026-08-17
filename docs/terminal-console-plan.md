# Terminal console plan

End in mind: a terminal-mode watch. Verbose Lina logs on the color OLED, date/time/Wi-Fi on a split bottom pane, triple-tap Wi-Fi, gray LCD still a clock when the OLED sleeps.

## Layout (not actual tmux)

400×400 round OLED, bezel margin so glyphs are not clipped. Font large, console/fastboot density (~18–22 px), not Asteroid QML.

- **Top ~70%:** faux `$ ` shell from the voice journal (not raw `watch-voice:` lines). Errors in red. 24h scrollback, phone-style up/down in the pane; top/bottom screen edges stay launcher swipes. New echo pins the last `$ ` to the bottom of the pane.
- **Bottom ~30%, top-aligned:** date left / time+TZ right, WIFI left / BAT right, English weekday centered under that. Hot-pink shell; blue and purple status.

Wi-Fi changes (triple-tap or rate-limit disable) log into the voice journal **and** update the bottom line.

## Drawing path

Preferred later: small Qt app, `QT_QPA_PLATFORM=hwcomposer`, no Wayland desktop.

**Now:** Lipstick stays. The two-pane console is watchface `018-lina-console.qml`. Install with `~/.local/share/watch-voice/install-term.sh`. Do not stop `asteroid-launcher` until an LPM compositor stub exists. See [display-and-lcd.md](display-and-lcd.md).

## Triple-tap

Watchface `MouseArea` while MCE is DisplayOn or DisplayDim. Three taps in 1.5 s toggle ConnMan wifi `powered`. Do not add a second reader of `/dev/input/event1`: Lipstick’s evdevtouch plugin already has `fts_ts`, and that helper never saw taps. `watch-voice.sh` logs `wifi on` / `wifi off` on Powered flips.

## Voice script

Minimal: log ConnMan wifi powered/connected flips. `wait_for_transport` already pauses until wifi is back. No extra sleep after rate-limit clips.

## Order (stop between steps for on-wrist tests)

1. Spike: stop launcher, prove MCE unblank **and** gray LCD. **Unblank works; gray LCD does not. Launcher restored.**
2. Resolve compositor LPM: **keep Lipstick**; replace the watchface only (QML-only terminal). Stub compositor is later.
3. Two-pane console watchface + `watch-term-feed.service` via `install-term.sh`. **Confirmed 2026-08-16:** color console + gray LCD on top key and idle timeout. App grid / swipes still exist.
4. Triple-tap Wi-Fi on the color console watchface (not a second `/dev/input/event1` reader — Lipstick already has the device and that helper never saw taps). **Confirmed 2026-08-16.**
5. Persistently disable `asteroid-launcher` only after gray LCD + console are proven **and** LPM works without Lipstick. Leave packages installed.
6. CPU `performance` @ 1.094 GHz is a later conversation.

## Watch-voice install reminder

Do not ship GitHub-only runtime. Install to the watch first. Never commit `groq.key`.
