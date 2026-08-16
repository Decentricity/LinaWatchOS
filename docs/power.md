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

## 2026-08-17: 70% → 44% on this USB cable

`nanohub_fuelgauge-0` is the real meter (MCE matches). Ignore the generic `battery` node.

This morning, not overnight:

- **04:05** wrist: 71% at 3.98 V, `charger_online=0`
- **04:11** first plug into this PC: **70%**. One-second `AC=+81 mA`, then `AC≈−100 mA` with `charger_online=1`
- **04:11–05:16** gadget up, MCE “charging”, cell still discharging: **70% → 49%**
- **05:20** unplug. **05:26** replug: same +82 mA blip, then **−120 to −240 mA**, pack **49% → 44%** in ~7 min, temperature up

MCE “charging” is the charger *bit*. Nanohub `AC` negative is net out of the cell. This host’s USB + RNDIS gadget does not cover Linux + Lipstick. Plug/unplug wakes the OLED and restarts the gadget; unplugged intervals charge nothing.

`watch-bottom-wifi` was idle in `evdev_read`. `watch-voice` and `watch-term-feed` were ~1% of one core. `asteroid-launcher` can peg **~1 full core with DisplayOff**. All four CPUs stay `performance` @ 1.094 GHz even though MCE says governor **automatic**.
