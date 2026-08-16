# Do not

These are hard constraints. They exist because we already paid for them.

## This PC

- Never reboot, power off, reset, suspend, or restart this computer as an agent.
- Never install Carbonyl (obsolete Chromium; this machine holds crypto assets).
- Never install third-party software unless the user named that product. Audit even then.
- This PC is installer/debug only. Lina does not run here.

## The watch display

- Never `dd` or raw-write `/dev/fb0`.
- Never force `/sys/class/leds/lcd-backlight/brightness` to 0 as an experiment.
- Unblank only through MCE (`mcetool --unblank-screen` or the **top** short-press).
- After earlier fb0/backlight experiments the color panel stayed dark until a **watch** reboot. A PC reboot will not fix it and must not be done.

## Nanohub / gray LCD

- Do not write `/sys/class/nanohub/nanohub/lock` (can brick nanohub).
- Do not use firmware `download*` / `erase_shared*` nodes.
- `lcd-tools` is the supported CLI. `lcd-tools --prepare-timepiece` is aggressive (cutOffScreen, bandMode) and returned errors without Lipstick.

## Voice loop regressions not to reintroduce

- USB as Groq transport
- Goodbye-kills-service
- Whisper `prompt=`
- Toybox `date` (use `/usr/bin/date`)
- USB as “connected” for the voice loop
- Flashlight cue
- Per-second VAD, settle/playback-RMS filters, `MIN_KEEP_S=8`
- GStreamer volume/wavenc on Pico
- BusyBox `awk srand` for clip picking (use `/dev/urandom`, never repeat last)
- Extra `sleep 0.5` after the listen cue
- Unapproved 60-second wait after rate-limit clips

## Buttons

- **Top short-press** is the daily wake / unblank. She has always used that.
- Bottom long-hold ~10s is PMIC hardware reset.
- Do not describe MCE `powerkey` as “she pressed bottom.”
