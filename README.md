# video-interpreter-harness-skill

A skill so agents (Claude, Codex, Cursor) can **actually watch videos**: `ffmpeg` extracts frames + audio, Whisper builds a transcript, and the model reads the images with timestamps — useful for screencasts, demos, and bug repros.

It does not make the model natively “see” video bytes; it gives a real shell pipeline.

## Prerequisites

- `ffmpeg` / `ffprobe` (`brew install ffmpeg`)
- Whisper CLI (optional, if the video has speech): `openai-whisper`, `faster-whisper`, etc.

## Install

### Claude Code

```bash
mkdir -p ~/.claude/skills/video-interpreter
cp skills/video-interpreter/SKILL.md ~/.claude/skills/video-interpreter/
```

Restart Claude. Use `/video-interpreter` or ask it to watch a video.

### Codex

```bash
mkdir -p ~/.codex/skills/video-interpreter/agents
cp skills/video-interpreter/SKILL.md ~/.codex/skills/video-interpreter/
cp skills/video-interpreter/agents/openai.yaml ~/.codex/skills/video-interpreter/agents/
```

Restart Codex. Use `/video-interpreter` or **Video Interpreter**.

### Cursor

Copy the skill into project or user skills:

```bash
# project
mkdir -p .cursor/skills/video-interpreter
cp skills/video-interpreter/SKILL.md .cursor/skills/video-interpreter/

# or global (if you use ~/.cursor/skills)
mkdir -p ~/.cursor/skills/video-interpreter
cp skills/video-interpreter/SKILL.md ~/.cursor/skills/video-interpreter/
```

In chat, ask it to follow the **video-interpreter** skill when analyzing a `.mp4`/`.mov`.

## Quick use

Point at a local video and ask for a summary (timestamps, steps, what broke). Short bug screencasts (~1–5 min) are the cost sweet spot.
