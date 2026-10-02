# video-interpreter-harness-skill

Skill para agentes (Claude, Codex, Cursor) **assistirem vídeos de verdade**: `ffmpeg` extrai frames + áudio, Whisper gera transcript, e o modelo lê as imagens com timestamps — útil pra screencasts, demos e bug repros.

Não faz o modelo “ver bytes de vídeo” nativamente; dá o pipeline técnico no shell.

## Pré-requisitos

- `ffmpeg` / `ffprobe` (`brew install ffmpeg`)
- Whisper CLI (opcional, se o vídeo tiver fala): `openai-whisper`, `faster-whisper`, etc.

## Instalar

### Claude Code

```bash
mkdir -p ~/.claude/skills/video-interpreter
cp skills/video-interpreter/SKILL.md ~/.claude/skills/video-interpreter/
```

Reinicie o Claude. Use `/video-interpreter` ou peça pra assistir um vídeo.

### Codex

```bash
mkdir -p ~/.codex/skills/video-interpreter/agents
cp skills/video-interpreter/SKILL.md ~/.codex/skills/video-interpreter/
cp skills/video-interpreter/agents/openai.yaml ~/.codex/skills/video-interpreter/agents/
```

Reinicie o Codex. Use `/video-interpreter` ou o nome **Video Interpreter**.

### Cursor

Copie a pasta para as skills do projeto ou do usuário:

```bash
# no projeto
mkdir -p .cursor/skills/video-interpreter
cp skills/video-interpreter/SKILL.md .cursor/skills/video-interpreter/

# ou global (se você usa ~/.cursor/skills)
mkdir -p ~/.cursor/skills/video-interpreter
cp skills/video-interpreter/SKILL.md ~/.cursor/skills/video-interpreter/
```

No chat, peça pra seguir a skill **video-interpreter** ao analisar um `.mp4`/`.mov`.

## Uso rápido

Aponte um vídeo local e peça o resumo (com timestamps, passos e o que quebrou). Vídeos curtos de bug (~1–5 min) são o sweet spot de custo.
