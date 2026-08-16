# Power and CPU

Measured 2026-08-16 while **USB-cradled, charging, OLED backlight 0**, Wi-Fi state mixed. Net `current_now` is charge into the cell, not wrist drain. Do not quote milliwatts from that.

## What is expensive

All four cores are **online, governor `performance`, locked at 1.094 GHz** even at ~90% idle. That is Asteroid/kernel, not the voice script. Left out of the first terminal plan unless asked.

`asteroid-launcher` (Lipstick compositor + desktop):

- ~**108 MB RSS** (about a quarter of RAM)
- ~1% of one core with the panel blanked
- Largest userspace CPU since boot (~176 s in ~3.7 h)

## What is cheap

`watch-voice.sh` (main loop + Wi-Fi watcher):

- ~**3 MB RSS**
- well under 1% of one core
- ~2 s of CPU in several minutes of running
- never in the top 25 CPU users

With the panel blank: `composer@2.1`, Pulse, MCE, `sensorfwd` were ~0% CPU. GPU `gpubusy` was 0.

Load average ~2.5 with 90% idle is D-state kernel waits (display/Wi-Fi), not 2.5 busy cores.

## Hypothesis

Decentricity’s hypothesis was that Lipstick, not Lina, is the hog. The samples agree on **RAM and always-on compositor**, not on the shell loop. Removing Lipstick is the userspace win. Gray LCD handover currently still needs *something* that owns `org.nemomobile.compositor` (see display notes).
