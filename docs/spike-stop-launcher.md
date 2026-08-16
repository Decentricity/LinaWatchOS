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

Launcher is **active** again as of this log. Confirm on-wrist that the gray clock is back after blank.

## Conclusion

Lipstick is not required to **unblank** the OLED or to run Lina. It **is** required today for MCE **LPM_ON**, which is the gray LCD handover. Next work is an LPM compositor stub or a Lipstick process that only hosts the terminal surface. Do not write `fb0`.
