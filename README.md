ï»¿# videomotival

**ê°Å¸â¡Â§ê°Å¸â¡Â· [PortuguêÂªs](README.md) êÂ· ê°Å¸â¡Âºê°Å¸â¡Â¸ [English](README.en.md) êÂ· ê°Å¸â¡Âªê°Å¸â¡Â¸ [EspaêÂ±ol](README.es.md)**

## ê°Å¸ââ Guia de uso

Guia completo (landing + passo a passo): **https://inematds.github.io/videomotival/guia/**

Canal motivacional automatizado, custo zero, rodando local:

```
TEMA ê¢â â HISTêâRIA (LLM) ê¢â â CENAS ê¢â â VOZ por cena (inemavox) ê¢â â ILUSTRAêâ¡êâ¢ES (flux2-klein) ê¢â â ffprobe ê¢â â FFmpeg ê¢â â MP4 16:9
```

ImplementaêÂ§êÂ£o do tutorial *"Build a $0 Motivational YouTube Channel Automation with Codex Skills"*
adaptada ao ecossistema desta mêÂ¡quina. O zip original do tutorial (skill Codex + `video_pipeline.py`)
estêÂ¡ preservado em `.agents/skills/motivational-story-video/` e `docs/tutorial-original/`.

TrêÂªs jeitos de usar, todos sobre o mesmo pipeline:

| Modo | Como |
|---|---|
| **Interface web** | `python3 -m videomotival serve` ê¢â â http://127.0.0.1:8030 |
| **CLI** | `python3 -m videomotival create --topic "ê¢â¬Â¦" --duration 75 --auto` |
| **Skill do Claude Code** | `.claude/skills/motivational-story-video/SKILL.md` ê¢â¬â "faz um vêÂ­deo motivacional de 90 s sobre X" |

## Requisitos

- `ffmpeg` / `ffprobe`, Python 3.12, Pillow (têÂ­tulo da cena 1), FastAPI + uvicorn (sêÂ³ para o web).
- **inemavox** rodando em `localhost:8010` (voz: `chatterbox` + `rachel` por padrêÂ£o; `edge` para testes sem GPU).
- **inemaimg** rodando em `localhost:8000` com `flux2-klein` (imagens 1920x1080, ~12 s cada).
- Uma chave de LLM para a histêÂ³ria, lida em runtime de `~/projetos/openpcbotv2/.env` ou `~/projetos/wifi/.env`
  (`OPENROUTER_API_KEY` ê¢â â `GROQ_API_KEY` ê¢â â fallback Ollama local). Nada de chave neste repo.

```bash
pip install -r requirements.txt
python3 -m videomotival check        # confere tudo sem imprimir segredos
```

## Fluxo (igual ao tutorial, etapa por etapa)

```bash
# 1ê¢â¬â3. tema ê¢â â roteiro + cenas (para em revisêÂ£o)
python3 -m videomotival create --topic "Pare de esperar a motivaêÂ§êÂ£o" --duration 75
#    ê¢â â ~/projetos/output/videomotival/<run-id>/work/{story.md,transcript.md,scenes.json}

# 4ê¢â¬â5. narraêÂ§êÂ£o por cena (001.mp3, 002.mp3ê¢â¬Â¦) + mediêÂ§êÂ£o ffprobe
python3 -m videomotival tts --run <run-id>

# 6. uma ilustraêÂ§êÂ£o por cena (001.png, 002.pngê¢â¬Â¦), personagem consistente
python3 -m videomotival images --run <run-id>

# 7. cena = imagem + êÂ¡udio com zoom lento; concat ê¢â â final.mp4
python3 -m videomotival render --run <run-id> [--music trilha.mp3]

# 8. tudo de uma vez, sem paradas
python3 -m videomotival create --topic "Por que o progresso pequeno vence a motivaêÂ§êÂ£o grande" --duration 90 --auto
```

Outros comandos: `build` (tts+images+render), `recompose` (depois de editar `scenes.json` êÂ  mêÂ£o: recompêÂµe
prompts e invalida sêÂ³ as cenas alteradas), `regen-image --scene N`, `status`, `list`, `voices`.
Cada etapa pula o que jêÂ¡ existe; `--force` refaz.

## Estrutura de uma corrida

```
~/projetos/output/videomotival/<timestamp>-<slug>/
ê¢âÅê¢ââ¬ê¢ââ¬ metadata.json        estado (status, progresso, erro) ê¢â¬â a web lêÂª daqui
ê¢âÅê¢ââ¬ê¢ââ¬ manifest.json        resultado verificado (1920x1080, duraêÂ§êÂ£o, cenas)
ê¢âÅê¢ââ¬ê¢ââ¬ final.mp4
ê¢ââê¢ââ¬ê¢ââ¬ work/
    ê¢âÅê¢ââ¬ê¢ââ¬ story.md, transcript.md, scenes.json, story_raw.txt, durations.json, log.txt
    ê¢âÅê¢ââ¬ê¢ââ¬ audio/001.mp3 ê¢â¬Â¦   images/001.png ê¢â¬Â¦   video/001.mp4 ê¢â¬Â¦
    ê¢ââê¢ââ¬ê¢ââ¬ title.png         têÂ­tulo da cena 1 (Pillow ê¢â â overlay ffmpeg)
```

## DecisêÂµes que diferem do tutorial

- **Voz**: inemavox (chatterbox/rachel) no lugar de Fish Audio. O provedor `fish` continua
  implementado (`TTS_PROVIDER=fish`, `FISH_VOICE_ID` no `.env`, `FISH_API_KEY` nos arquivos de segredo).
- **Imagens**: flux2-klein local no lugar de "Codex image generation". ConsistêÂªncia do personagem =
  bêÂ­blia verbatim em todo prompt + seed fixa por corrida (sem imagem de referêÂªncia).
- **TêÂ­tulo da cena 1**: gravado pelo ffmpeg a partir de PNG do Pillow (texto exato), nêÂ£o pedido ao
  gerador de imagem (flux êÂ© fraco em letras; o drawtext do ffmpeg 6.1 corta acentos).
- **PortêÂ£o de duraêÂ§êÂ£o**: `DURATION_GATE=warn` por padrêÂ£o (o tutorial recusa renderizar fora de êÂ±8 %).
  O wpm medido êÂ© gravado em `output/videomotival/calibration.json` e realimenta o roteirista.
- **Idioma**: `pt` por padrêÂ£o; `--lang en` reproduz o tutorial.
- **Respiro**: 0,45 s de silêÂªncio ao fim de cada narraêÂ§êÂ£o (`AUDIO_PAD_SECONDS`).
- **MêÂºsica de fundo** opcional na montagem (`--music`, -18 dB aprox., fade).

## ConfiguraêÂ§êÂ£o

Copie `.env.example` para `.env` e ajuste. Tudo ali êÂ© nêÂ£o secreto (provedores, voz, urls, portas).
VariêÂ¡veis de ambiente do processo vencem o arquivo.

## VersêÂ£o

`videomotival/__init__.py` ê¢â â `1.0.0`.

---

Projeto  - ARCKHÉe