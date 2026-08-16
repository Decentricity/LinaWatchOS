# Power and CPU

Measured 2026-08-16 while USB-cradled, OLED backlight 0, Wi-Fi state mixed. Nanohub `AC` / sysfs `current_now` is **net cell current**. Negative `AC` this morning meant the pack was discharging even when MCE said charging. Do not quote milliwatts from the generic `battery` node.

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

## 2026-08-17: 70% → 44% this morning on this PC USB

`nanohub_fuelgauge-0` is the real meter (MCE matches). Ignore the generic `battery` node.

Two things at once. Neither is “the cradle is broken”:

1. **The watch is drawing a lot with the OLED off.** For most of 04:11–05:16, nanohub `AC` was about **−100 mA** (worse, **−120 to −240 mA**, during the 05:26 debug window). That is wrist-watch current, not phone current. `asteroid-launcher` can peg ~1 full core at DisplayOff. All four CPUs stay `performance` @ 1.094 GHz even though MCE says governor **automatic**. Lina’s helpers are not that load (`watch-bottom-wifi` idle; voice/feed ~1% of one core).
2. **This PC USB + RNDIS does not outrun that draw.** The charger *bit* is on (`charger_online=1`, MCE “charging”). The cell still goes down. Plug-in is a one-second `AC=+80 mA` blip, then back to net discharge. Unplug/replug wakes the OLED and restarts the gadget; unplugged intervals add nothing. A wall charger or the stock dock is not what we measured.

Timeline:

- **04:05** not on this PC: 71% at 3.98 V, `charger_online=0`
- **04:11** first plug here: **70%**
- **04:11–05:16** gadget up: **70% → 49%** at ~−100 mA
- **05:20** unplug. **05:26** replug: **49% → 44%** in ~7 min, temperature up

