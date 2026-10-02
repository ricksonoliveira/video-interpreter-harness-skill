---
name: video-interpreter
description: >-
  Use when the user attaches, drops, or points to a video (screen recording,
  demo, bug repro, tutorial) and wants it actually watched via shell tools —
  extract frames + transcript, then reason over them. Not for conceptual
  reminders that videos exist.
---

# Video Interpreter (technical path)

## Goal
Give a real pipeline to “watch” a video: probe → extract timed frames → transcribe audio → read the images + transcript → answer. Do **not** invent what is on screen.

## Prerequisites (install if missing)
```bash
ffmpeg -version
ffprobe -version

whisper --help          # openai-whisper
# or: faster-whisper / mlx-whisper / a hosted speech-to-text CLI
```
If `ffmpeg` is missing: install via the OS package manager (`brew install ffmpeg`, `apt install ffmpeg`, etc.) before continuing.

## Inputs
- Local path to `.mp4` / `.mov` / `.webm` / `.mkv`, or a URL you download first to a local file.
- Work dir: create `./video-watch/<basename>/` next to the file (or under the agent workspace). Keep all artifacts there.

## Step 1 — Probe
```bash
ffprobe -v error -show_entries format=duration,size -show_entries stream=codec_type,codec_name,width,height,r_frame_rate,sample_rate -of json "$VIDEO"
```
Record: duration, resolution, has-video / has-audio. Cap work for very long videos (see Tuning).

## Step 2 — Extract frames (prefer scene change + density floor)
**Screen recordings / bug repros** (subtle UI changes): lower scene threshold and keep a time floor so slow changes still get sampled.

```bash
OUT="./video-watch/$(basename "$VIDEO" | sed 's/\.[^.]*$//')"
mkdir -p "$OUT/frames"

ffmpeg -y -i "$VIDEO" \
  -vf "select='gt(scene\\,0.25)+isnan(prev_selected_t)+gte(t-prev_selected_t\\,2)',scale=min(1280\\,iw):-2,showinfo" \
  -vsync vfr -q:v 3 "$OUT/frames/%04d.jpg" \
  2> "$OUT/ffmpeg-frames.log"
```

Fallback if too few frames (< ~8 for a multi-minute screencast), or motion itself matters:
```bash
ffmpeg -y -i "$VIDEO" -vf "fps=1,scale=min(1280\\,iw):-2" -q:v 3 "$OUT/frames/%04d.jpg"
```

Hard cap: keep at most **~60–80** frames for a single analysis pass. If over the cap, raise the scene threshold / fps interval, or sample evenly across the timeline. Deduplicate near-identical consecutive frames when obvious.

Build a **manifest** `$OUT/manifest.jsonl` mapping `file → approximate timestamp`. Parse `pts_time` from `showinfo` in the log when available; for `fps=N` mode, timestamp ≈ `(index-1)/N` seconds.

## Step 3 — Audio → transcript
```bash
ffmpeg -y -i "$VIDEO" -vn -ac 1 -ar 16000 -c:a pcm_s16le "$OUT/audio.wav"
```
Then transcribe (example with Whisper CLI):
```bash
whisper "$OUT/audio.wav" --model base --output_format json --output_dir "$OUT"
# Prefer language flag when known: --language pt / en
```
If the video has a sidecar `.srt`/`.vtt` or an embedded subtitle stream, use that instead of re-transcribing. Silent UI-only screencasts: skip Whisper; rely on frames.

## Step 4 — Actually “watch” (vision + transcript)
1. Read the transcript (with timestamps if present).
2. Open / attach frames **in chronological order**, each labeled with its timestamp from the manifest (e.g. `frame_0012.jpg @ 00:41`).
3. First pass: describe what happens over time; call out UI, errors, clicks, and anything ticket-worthy. **Cite timestamps.**
4. Never invent dialogue that is not in the transcript. Never invent UI text you did not read from a frame.
5. For follow-ups (“what happens at 1:20?”, “copy the error string”), re-open only the relevant frames / transcript slice — don’t re-extract unless needed.

## Step 5 — Deliver
Lead with what happened. Then: broken behavior, exact steps visible, environment clues, suggested bug title. Offer a ticket draft only if asked (or already requested).

## Tuning cheat sheet
| Footage | Approach |
| --- | --- |
| Screencast / bug repro | `scene≈0.2–0.3` + ≥1 frame / 2s; read UI text off frames |
| Edited / hard cuts | `scene≈0.3–0.4` |
| Talking head + slides | transcript-first; fewer frames at slide changes |
| Long (>15–20 min) | chunk by time (e.g. 5 min), analyze chunks, then summarize |

## Failure modes
- No `ffmpeg`: install it; don’t pretend you watched.
- Corrupt / unsupported codec: remux or re-encode once (`ffmpeg -i in -c copy out.mp4` or re-encode h264), then retry.
- Vision can’t read tiny text: re-extract a crop or higher-res still around that timestamp (`ffmpeg -ss T -i "$VIDEO" -frames:v 1 -q:v 2 still.jpg`).
- Empty audio / music-only: say so; don’t invent narration.

## Out of scope
- Claiming the model “natively” sees video bytes without this pipeline.
- Baking one user’s drop-folder path or ticket tracker into this skill.

## Token cost (be deliberate)
Vision tokens scale with **how many frames** you send into the model, not with video file size on disk. Prefer scene-aware extraction + hard caps (≤60–80, often far fewer for short screencasts). Send transcript as text (cheap). For long videos, chunk and summarize instead of dumping every frame into one prompt. Re-asks should reuse existing frames, not re-extract and re-attach everything.
