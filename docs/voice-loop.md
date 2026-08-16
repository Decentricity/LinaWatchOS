# Voice loop (watch-voice)

Source: `https://github.com/Decentricity/watch-voice`  
Local: `~/.local/share/watch-voice`  
On watch: `/var/lib/watch-voice/watch-voice.sh` via `watch-voice.service`

## Loop

1. Two-note listen chime (`cue.wav`, E5 then B5, plucked)
2. Continuous 6 s `tinycap`
3. RMS gate 25 (do not lower; that ignored the room AC for hours)
4. Groq STT `whisper-large-v3-turbo`
5. Groq chat `llama-3.3-70b-versatile` (Lina system prompt; time/battery are watch status, not her words)
6. Soft ack bell (`ack.wav`, C5) only after a real chat reply
7. Pico `en-US` TTS, play WAV as written, mixer **96** (cues/ack stay **84**)

Empty/junk transcripts (including Whisper fakes like “thank you”) → listen again, no ack.

## Wi-Fi as the pause signal

- Loop pauses while Wi-Fi is down (`wait_for_transport`).
- Wi-Fi off ~3 s: wipe history, play `wipe.wav` (falling sweep, former 220 Hz double beep).
- Idle 1 h also wipes. History cull ≥ 30 min.
- Rate-limit (`rate_limit_exceeded` on STT or chat): play one non-repeating apology clip, then one non-repeating tired clip (`/dev/urandom`, never last index), then `connmanctl disable wifi` **once**. Does not keep forcing it off. User can turn Wi-Fi on and try again.

## Sounds

| File | Role |
|---|---|
| `cue.wav` | Listen |
| `ack.wav` | Heard a real reply |
| `wipe.wav` | Memory wipe / disconnect |
| `sorry1–5.wav` | Rate-limit series A |
| `tired1–5.wav` | Rate-limit series B (tired5 wording is exactly “I need to can't speak right now.”) |

Regenerate: `generate-cue.sh`, `generate-limit.sh` (Pico on the PC).

## Mixer (speaker)

`PRI_MI2S_RX Audio Mixer MultiMedia1=1`, `RX3 MIX1 INP1=RX1`, `SPK=Switch`, `Ext Spk Switch=On`, `Speaker Boost=ENABLE`.

Capture: DMIC1 / `MultiMedia1 Mixer TERT_MI2S_TX=1`.

## Pico on the watch

- `/var/lib/watch-voice/pico/bin/pico2wave` (armhf)
- libs in `pico/lib`
- lang bins also copied to `/usr/share/pico/lang/` (hardcoded path)
- `pico2wave` refuses output names that do not end in `.wav`
