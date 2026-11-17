# videomotival

**ðŸ‡§ðŸ‡· [PortuguÃªs](README.md) Â· ðŸ‡ºðŸ‡¸ [English](README.en.md) Â· ðŸ‡ªðŸ‡¸ [EspaÃ±ol](README.es.md)**

## ðŸ“– Guia de uso

Guia completo (landing + passo a passo): **https://inematds.github.io/videomotival/guia/**

Canal motivacional automatizado, custo zero, rodando local:

```
TEMA â†’ HISTÃ“RIA (LLM) â†’ CENAS â†’ VOZ por cena (inemavox) â†’ ILUSTRAÃ‡Ã•ES (flux2-klein) â†’ ffprobe â†’ FFmpeg â†’ MP4 16:9
```

ImplementaÃ§Ã£o do tutorial *"Build a $0 Motivational YouTube Channel Automation with Codex Skills"*
adaptada ao ecossistema desta mÃ¡quina. O zip original do tutorial (skill Codex + `video_pipeline.py`)
estÃ¡ preservado em `.agents/skills/motivational-story-video/` e `docs/tutorial-original/`.

TrÃªs jeitos de usar, todos sobre o mesmo pipeline:

| Modo | Como |
|---|---|
| **Interface web** | `python3 -m videomotival serve` â†’ http://127.0.0.1:8030 |
| **CLI** | `python3 -m videomotival create --topic "â€¦" --duration 75 --auto` |
| **Skill do Claude Code** | `.claude/skills/motivational-story-video/SKILL.md` â€” "faz um vÃ­deo motivacional de 90 s sobre X" |

## Requisitos

- `ffmpeg` / `ffprobe`, Python 3.12, Pillow (tÃ­tulo da cena 1), FastAPI + uvicorn (sÃ³ para o web).
- **inemavox** rodando em `localhost:8010` (voz: `chatterbox` + `rachel` por padrÃ£o; `edge` para testes sem GPU).
- **inemaimg** rodando em `localhost:8000` com `flux2-klein` (imagens 1920x1080, ~12 s cada).
- Uma chave de LLM para a histÃ³ria, lida em runtime de `~/projetos/openpcbotv2/.env` ou `~/projetos/wifi/.env`
  (`OPENROUTER_API_KEY` â†’ `GROQ_API_KEY` â†’ fallback Ollama local). Nada de chave neste repo.

```bash
pip install -r requirements.txt
python3 -m videomotival check        # confere tudo sem imprimir segredos
```

## Fluxo (igual ao tutorial, etapa por etapa)

```bash
# 1â€“3. tema â†’ roteiro + cenas (para em revisÃ£o)
python3 -m videomotival create --topic "Pare de esperar a motivaÃ§Ã£o" --duration 75
#    â†’ ~/projetos/output/videomotival/<run-id>/work/{story.md,transcript.md,scenes.json}

# 4â€“5. narraÃ§Ã£o por cena (001.mp3, 002.mp3â€¦) + mediÃ§Ã£o ffprobe
python3 -m videomotival tts --run <run-id>

# 6. uma ilustraÃ§Ã£o por cena (001.png, 002.pngâ€¦), personagem consistente
python3 -m videomotival images --run <run-id>

# 7. cena = imagem + Ã¡udio com zoom lento; concat â†’ final.mp4
python3 -m videomotival render --run <run-id> [--music trilha.mp3]

# 8. tudo de uma vez, sem paradas
python3 -m videomotival create --topic "Por que o progresso pequeno vence a motivaÃ§Ã£o grande" --duration 90 --auto
```

Outros comandos: `build` (tts+images+render), `recompose` (depois de editar `scenes.json` Ã  mÃ£o: recompÃµe
prompts e invalida sÃ³ as cenas alteradas), `regen-image --scene N`, `status`, `list`, `voices`.
Cada etapa pula o que jÃ¡ existe; `--force` refaz.

## Estrutura de uma corrida

```
~/projetos/output/videomotival/<timestamp>-<slug>/
â”œâ”€â”€ metadata.json        estado (status, progresso, erro) â€” a web lÃª daqui
â”œâ”€â”€ manifest.json        resultado verificado (1920x1080, duraÃ§Ã£o, cenas)
â”œâ”€â”€ final.mp4
â””â”€â”€ work/
    â”œâ”€â”€ story.md, transcript.md, scenes.json, story_raw.txt, durations.json, log.txt
    â”œâ”€â”€ audio/001.mp3 â€¦   images/001.png â€¦   video/001.mp4 â€¦
    â””â”€â”€ title.png         tÃ­tulo da cena 1 (Pillow â†’ overlay ffmpeg)
```

## DecisÃµes que diferem do tutorial

- **Voz**: inemavox (chatterbox/rachel) no lugar de Fish Audio. O provedor `fish` continua
  implementado (`TTS_PROVIDER=fish`, `FISH_VOICE_ID` no `.env`, `FISH_API_KEY` nos arquivos de segredo).
- **Imagens**: flux2-klein local no lugar de "Codex image generation". ConsistÃªncia do personagem =
  bÃ­blia verbatim em todo prompt + seed fixa por corrida (sem imagem de referÃªncia).
- **TÃ­tulo da cena 1**: gravado pelo ffmpeg a partir de PNG do Pillow (texto exato), nÃ£o pedido ao
  gerador de imagem (flux Ã© fraco em letras; o drawtext do ffmpeg 6.1 corta acentos).
- **PortÃ£o de duraÃ§Ã£o**: `DURATION_GATE=warn` por padrÃ£o (o tutorial recusa renderizar fora de Â±8 %).
  O wpm medido Ã© gravado em `output/videomotival/calibration.json` e realimenta o roteirista.
- **Idioma**: `pt` por padrÃ£o; `--lang en` reproduz o tutorial.
- **Respiro**: 0,45 s de silÃªncio ao fim de cada narraÃ§Ã£o (`AUDIO_PAD_SECONDS`).
- **MÃºsica de fundo** opcional na montagem (`--music`, -18 dB aprox., fade).

## ConfiguraÃ§Ã£o

Copie `.env.example` para `.env` e ajuste. Tudo ali Ã© nÃ£o secreto (provedores, voz, urls, portas).
VariÃ¡veis de ambiente do processo vencem o arquivo.

## VersÃ£o

`videomotival/__init__.py` â†’ `1.0.0`.

---

Projeto adaptado da comunidade INEMA TDS - Marca Arckhe