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

## 2026-08-17: 49% is the real pack

After ~12.5 h this boot, `nanohub_fuelgauge-0` was **49% at 3.53 V**, charging on USB at ~200 mA. MCE matches. This is not the lying generic `battery` node (that one happened to say 50%). 3.53 V is a mid-to-low cell; the worn gauge’s full is still ~243 mAh vs 415 mAh design.

A drop from full to half overnight is **Asteroid’s baseline**, not the new helpers:

- All four CPUs stay on `performance` at 1.094 GHz. MCE “CPU Scaling Governor: automatic” does not change that.
- `asteroid-launcher` can peg **~1 full core with DisplayOff** (about 25% of the SoC). That is the compositor, not the watchface 1 Hz timer (`running: oledLive` only).
- `watch-bottom-wifi` was idle in `evdev_read` (0 jiffies). `watch-voice` and `watch-term-feed` were ~1% of one core each.
- USB gadget can charge and SSH at the same time; a 20–30 min debug session on the cradle does not explain 50 points of SoC.

~10 mA average off-charger for 12.5 h would land a 243 mAh pack near 50%. Daily charging is still enough. The long-term win is still Lipstick / the locked `performance` governor, not the voice loop.
