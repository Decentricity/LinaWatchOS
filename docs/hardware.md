# Hardware and access

## Watch

- **Device:** TicWatch Pro 2018, AsteroidOS 2.1, codename `catfish`
- **Kernel:** Linux 3.18.120 armv7l, 4 CPUs
- **RAM:** ~469 MB
- **Color panel:** AMOLED, `/dev/fb0` `mdssfb_90000`, mode `U:400x400p-59`, 32 bpp, virtual 400x1200
- **Gray panel:** FSTN LCD driven by **nanohub**, not by the framebuffer. Segmented clock (time, date, steps, battery icons). Can run while Linux is asleep or off.
- **Touch:** `fts_ts` → `/dev/input/event1`
- **Keys:** `qpnp_pon` event0 (top/power), `gpio-keys` event2 (bottom; Asteroid does not really support it)
- **Audio:** TinyALSA `tinycap` / `tinyplay` / `tinymix` on card 0. PulseAudio is not required for Lina.
- **Battery sysfs:** `/sys/class/power_supply/nanohub_fuelgauge-0/` (the voice script uses this). Android `battery` capacity can disagree; ignore it for Lina’s status line.

## SSH

| Path | Address | When |
|---|---|---|
| USB RNDIS | `root@192.168.2.15` | Watch in the cradle |
| Wi-Fi | ConnMan address (often `192.168.0.181`) | Wi-Fi on |

USB gadget stays up if Wi-Fi is disabled (`connmanctl disable wifi`). That is how we still debug after the rate-limit one-shot Wi-Fi off.

## Users on the watch

- `root` — ssh, systemd system units, `watch-voice.service`
- `ceres` (uid 1000) — Asteroid session: Lipstick, lcd-tools user timers, Pulse, boosters

## Graphics stack (hybris)

- `android-init.service` → `/system/bin/init`
- `android.hardware.graphics.composer@2.1-service`
- `android.hardware.graphics.allocator@2.0-service`
- Qt `qt5-qpa-hwcomposer-plugin` is installed
- Kernel fbcon is **not** usable: `vtcon0` is a dummy device. Fastboot-looking text must be drawn in userspace.
