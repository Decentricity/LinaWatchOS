# Spike: stop asteroid-launcher (2026-08-16)

## Setup

- Watch cradled, USB SSH `root@192.168.2.15`
- `asteroid-launcher` was active under `ceres`
- Display was off, backlight 0

## Action

```
systemctl --user -M ceres@ stop asteroid-launcher.service
```

Not disabled, not uninstalled.

## Results

| Check | Result |
|---|---|
| Launcher | inactive |
| `mcetool --unblank-screen` | `Display state: on`, backlight 76 |
| Color OLED | Black panel (nothing drawing). Expected. |
| `mcetool --blank-screen` | `Display state: off`, backlight 0 |
| Unblank again | on, backlight 76 |
| hwcomposer / allocator / mce | still running |
| `lcd-tools --sync-time` | ran (linker warnings only) |
| **Gray FSTN after blank** | **No time.** User confirmed. |
| Top short-press while launcher down | MCE requested `LPM_ON`, then `no compositor; going to logical off` |

## Follow-up probes (still without Lipstick)

- `mcetool -E enabled` — setting took; did not restore LCD.
- `--blank-screen-lpm` while already off — **denied** (`display is off`). Need ON → LPM_ON.
- Unblank then `--blank-screen-lpm` — MCE: `The name org.nemomobile.compositor was not provided by any .service files`; still logical off.
- `lcd-tools --prepare-timepiece` — `autoLowPowerScreen: 4`, `bandMode: 4`. Not a substitute.

## Rollback

```
systemctl --user -M ceres@ start asteroid-launcher.service
```

Launcher came back. Gray FSTN clock returned. Then **buttons appeared dead**: top short-press did not visibly take over from the gray LCD.

Cause: during the spike, `mcetool -E enabled` was used, then LPM was left **`disabled`** while `Powerkey blanking mode` was still `lpm`. MCE still logged powerkey (unblank ON, then `LPM_ON` → immediate `OFF`). The gray nanohub LCD stayed up, so it looked like the keys did nothing. OLED unblank was ~1 s of black behind the LCD.

Fix applied 16:48 WIB: `mcetool -E enabled` then `mcetool --unblank-screen`. Display **on**, backlight 76, LPM enabled. User still reported **only the gray LCD** (16:49). MCE/backlight were already ON; the FSTN layer was still in timepiece.

Follow-up 16:51: restarted Lipstick (compositor name had dropped from the session bus). 16:52: `setAmbientEnabled false`, `setUpdatesEnabled true`, MCE brightness 80 (sysfs 204). OLED should be bright if the panel is actually emitting; if the gray clock is still the only thing visible, nanohub LCD was left in timepiece by `--prepare-timepiece` and needs `nativeEnableLowPowerScreen(false)` or a **watch** reboot (not this PC).

## Conclusion

Lipstick is not required to **unblank** the OLED or to run Lina. It **is** required today for MCE **LPM_ON**, which is the gray LCD handover. Next work is an LPM compositor stub or a Lipstick process that only hosts the terminal surface. Do not write `fb0`.
