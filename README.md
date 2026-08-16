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
| Stop Lipstick spike | OLED unblanks without Lipstick. Gray LCD does **not**. Launcher restored. After restore, LPM was left disabled so keys looked dead; LPM re-enabled and OLED unblanked. |
| Two-pane OLED console | Watchface `018-lina-console.qml` (Lipstick stays). Color console + gray LCD handover confirmed 2026-08-16. |
| Triple-tap Wi-Fi | Watchface MouseArea (Lipstick owns `fts_ts`; a second evdev reader sees no events). OLED on/dimmed only. |

Lipstick stays until there is an LPM stub. Do not `dd` `/dev/fb0`. Do not `--prepare-timepiece`.

## Docs

- [Hardware and access](docs/hardware.md)
- [Do not](docs/do-not.md)
- [Power and CPU](docs/power.md)
- [Voice loop](docs/voice-loop.md)
- [Display, MCE, gray LCD](docs/display-and-lcd.md)
- [Keep vs drop](docs/keep-vs-drop.md)
- [Terminal console plan](docs/terminal-console-plan.md)
- [Spike log](docs/spike-stop-launcher.md)
- [Console watchface](docs/lina-console-watchface.md)

## Related

- Voice source/installer: `https://github.com/Decentricity/watch-voice`
- Local tree: `~/.local/share/watch-voice`
- Watch USB SSH: `root@192.168.2.15`
- Watch Wi-Fi SSH: ConnMan address (often `192.168.0.181`)
