# Keep vs drop

Goal: Linux framework, touch, graphics (only enough for a small console), SSIDs, OLED↔gray LCD handover. No Asteroid launcher/desktop.

## Keep

| Piece | Why |
|---|---|
| systemd, kernel, sshd, usb-moded | Access |
| android-init, hwcomposer, allocator | Graphics HAL; Qt hwcomposer QPA |
| mce, dsme | Top short-press, double-tap wake, dim/blank, ALS |
| lcd-tools + lcd-sync-time.timer | Time sync to nanohub |
| nanohub (untouched lock/download) | Gray LCD + fuel gauge + sensors |
| connman, wpa_supplicant | SSIDs; enable/disable wifi |
| sensorfwd | Tilt-to-wake via MCE sensor-gestures |
| TinyALSA + watch-voice.service | Lina |
| `org.nemomobile.compositor` **or equivalent LPM ack** | Gray LCD on blank (see display notes) |

## Stop first, do not uninstall yet

Rollback is `systemctl --user -M ceres@ start asteroid-launcher`.

- `asteroid-launcher.service` (**do not stop** until LPM stub exists; QML-only terminal is a watchface on Lipstick)
- asteroid-* apps (calculator, settings, …)
- mapplauncherd boosters
- PulseAudio (Lina uses tinyplay)
- ngfd, dconf, profiled, mpris-proxy
- asteroid-btsyncd
- ofono unless we need telephony later

## Input policy (agreed)

- Double-tap and top short-press stay with MCE (wake OLED).
- **Triple-tap toggles Wi-Fi only while the color OLED is already on.** Does not fight double-tap wake.
- One-shot `connmanctl enable|disable wifi`, same as rate-limit disable. Do not keep forcing Wi-Fi off.
