# Agent notes

This tree is **documentation** for LinaWatchOS. Do not treat it as the voice runtime.

- Voice runtime source: `~/.local/share/watch-voice` → GitHub `Decentricity/watch-voice`.
- After voice runtime edits: install to the watch first, then commit/push watch-voice. Never commit `groq.key`.
- After notes edits here: commit and push `Decentricity/LinaWatchOS` only. There is nothing to install from this repo yet.
- Never reboot, power off, or suspend the **PC**. A **watch** reboot is allowed if the user agrees.
- Never install Carbonyl. Never `dd` `/dev/fb0` or force `lcd-backlight` to 0. Never write nanohub `lock` / firmware `download*` files.
- Top short-press is the daily OLED wake. Do not tell the user it is the bottom button.
- Stop between user-visible steps so she can test on-wrist.

Read `docs/do-not.md` and `docs/display-and-lcd.md` before touching display or Lipstick.
