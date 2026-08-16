# LinaWatchOS

Notes and architecture for turning a TicWatch Pro 2018 (`catfish`, AsteroidOS 2.1) into a **terminal-mode watch**: Linux stays, Lipstick goes, Lina’s voice loop is the foreground.

This repository is documentation. Runtime code for the voice loop lives in [watch-voice](https://github.com/Decentricity/watch-voice). The watch itself is the runtime; the PC is installer and notes only.

**Owner:** Decentricity (Ms. Pandu Sastrowardoyo). The watch persona is Lina.

## Status (2026-08-16)

| Piece | State |
|---|---|
| Voice loop on watch | Running (`watch-voice.service`) |
| Pico TTS, chimes, rate-limit clips | Shipped in watch-voice |
| Rate-limit → speak clips → Wi-Fi off once | Shipped |
| Stop Lipstick spike | Done. OLED still unblanks via MCE. **Gray FSTN LCD does not show time** without Lipstick. Launcher restored. |
| Two-pane OLED console | Not built (blocked on LCD handover) |
| Triple-tap Wi-Fi | Not built |

Next decision: a tiny `org.nemomobile.compositor` stub that acks MCE LPM, **or** keep Lipstick and replace its QML with the terminal. Do not `dd` `/dev/fb0`.

## Docs

- [Hardware and access](docs/hardware.md)
- [Do not](docs/do-not.md)
- [Power and CPU](docs/power.md)
- [Voice loop](docs/voice-loop.md)
- [Display, MCE, gray LCD](docs/display-and-lcd.md)
- [Keep vs drop](docs/keep-vs-drop.md)
- [Terminal console plan](docs/terminal-console-plan.md)
- [Spike log](docs/spike-stop-launcher.md)

## Related

- Voice source/installer: `https://github.com/Decentricity/watch-voice`
- Local tree: `~/.local/share/watch-voice`
- Watch USB SSH: `root@192.168.2.15`
- Watch Wi-Fi SSH: ConnMan address (often `192.168.0.181`)
