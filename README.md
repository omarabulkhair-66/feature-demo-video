# feature-demo-video

Forked from [better-futures-studio/feature-demo-video](https://github.com/better-futures-studio/feature-demo-video). This fork renames the skill to `record-demo` and adds Egyptian Arabic narration through ElevenLabs. The MIT licence and its original copyright stay as they were.

A Claude Code and Codex skill for recording "developer shows the team a feature" videos of any web app running locally. The agent plans the narration with you, generates a casual voiceover on your Mac, drives the real app in a browser at a human pace (phone and desktop), and composes 1080p videos in light and dark.

The voice is cloned locally with Qwen3-TTS (free, nothing leaves your machine). It uses your own voice if you record a sample, otherwise a bundled default voice.

## Requirements

- macOS on Apple Silicon (the voice model runs on MLX)
- Node 22+, Python 3, `ffmpeg`, `uv`: `brew install ffmpeg uv` (on a fresh Mac, `python3` first prompts to install the Xcode Command Line Tools)
- [agent-browser](https://www.npmjs.com/package/agent-browser) 0.38+: `npm i -g agent-browser && agent-browser install`
- Optional: Tailscale, to watch videos on your phone

## Install

### Claude Code

As a plugin (updates through `/plugin`):

```
/plugin marketplace add omarabulkhair-66/feature-demo-video
/plugin install record-demo@omarabulkhair
```

Or as a plain skill, with the script below.

### Codex (and Claude Code without the plugin)

```bash
git clone https://github.com/omarabulkhair-66/feature-demo-video.git ~/.local/share/feature-demo-video
```

```bash
~/.local/share/feature-demo-video/install.sh
```

That links the skill into `~/.claude/skills` and `~/.codex/skills` (or `$CODEX_HOME/skills`). Pass `--claude` or `--codex` for just one. Pick either the plugin or the script for Claude Code, not both. To update, `git pull` in the clone.

### Voice model (once per Mac)

```bash
bash ~/.local/share/feature-demo-video/skills/record-demo/scripts/setup-local-voice.sh
```

It makes a Python environment under `~/.cache/feature-demo-video` and downloads about 5 GB of models. If you installed the plugin, ask the agent to run this; it knows where the skill lives.

## Use

In your project, ask the agent something like "record a demo video of the new export feature for the team". It will:

1. Read the feature's code and show you a beat-by-beat script to approve.
2. Work out with you how to run the app against a throwaway local database with fake data, with every external service mocked or switched off. It won't record against production or real credentials.
3. Generate the voice, record each segment, and compose light and dark videos into `/tmp/feature-demo/share/`.

## Your own voice

Ask the agent to "clone my voice for demos". Record 15–30 seconds of yourself talking at your normal demo pace (iPhone Voice Memos in a quiet room is fine) and give it the file. It trims the clip, transcribes it locally, and asks you to check the transcript. The sample is kept in `~/.config/feature-demo-video/voice/` on your Mac only. Delete that folder to go back to the default voice.

Only clone your own voice. Never commit a voice sample or share one: anyone holding it can make speech in your voice.

## Arabic voice (ElevenLabs)

Qwen3-TTS reads Arabic badly, so Arabic narration (Egyptian or Saudi) goes through ElevenLabs instead (cloud, costs credits). Every other language stays on the free local voice, and each script refuses the other's language.

1. Put your key in `~/.config/feature-demo-video/elevenlabs.env` (`chmod 600`), as one line: `ELEVENLABS_API_KEY=...`. An `ELEVENLABS_API_KEY` environment variable wins over the file.
2. Write the lines in Arabic, in the dialect you chose. The model is `eleven_v3`, which reads emotion tags in the text: `[excited]`, `[laughs]`, `[whispers]`.
3. Run `node skills/record-demo/scripts/voice-eleven.mjs <take-dir>`. It writes the same `vo/<id>.wav` and `lines.json` as the local voice, so recording and composing don't change.

The agent asks which dialect you want, Egyptian or Saudi, then lets you pick a voice, recommending the default. They're listed in `skills/record-demo/assets/eleven-voices.json`. Egyptian: male `masry` (default), `haytham`, `hanafi`; female `yasmine`, `fatima`, `nadia`. Saudi: male `hasawi` (default), `fahad`, `ziyad`; female `hana`. Choose with `ELEVENLABS_VOICE=hana`, take a dialect's default with `ELEVENLABS_DIALECT=saudi`, or pass any voice id with `ELEVENLABS_VOICE_ID`. Other settings: `ELEVENLABS_LANG` (default `ar`), `ELEVENLABS_MODEL`, `STABILITY` (v3 takes 0, 0.5 or 1; lower is more expressive), `TEMPO`, `REGEN=B,C` to re-take single lines.

## Known issues

- **Nobody has heard these voices outside the maintainer's one listening test.** Dialect, tone and emotion tags were checked on two short lines only, and the Saudi voices and wording rules not at all. Audition a full narration as audio before you record video.
- **The voice ids come from ElevenLabs' shared Voice Library.** Their owners can retire or change them, and library access through the API may depend on your plan. If a line fails with a 4xx error, swap the voice in `eleven-voices.json`; nothing is saved to `lines.json` until every line succeeds.
- **`eleven_v3` drifts a little in tone from clip to clip** and doesn't accept neighbouring-line context. Re-take the odd line with `REGEN`, or lower `STABILITY` on a flat one. With `eleven_multilingual_v2` the tags are stripped, because it would read them aloud.
- **A script is "Arabic" if any line has an Arabic letter.** A mixed Arabic/English script is voiced entirely by ElevenLabs and refused by the local voice. English product names inside Arabic lines may be read with an Arabic accent.
- **Words-per-minute is approximate for Arabic** (spaces split words, tags excluded); it only matters if you set `MAX_WPM`.
- **The key is a plain file** (mode 600) on your Mac. Don't put it in a repo, a `DEMO_DIR` or a chat.
- **Renaming broke old installs.** The skill and plugin are now `record-demo`. Remove the old `~/.claude/skills/feature-demo-video` and `~/.codex/skills/feature-demo-video` links, then run `install.sh` again; plugin users reinstall `record-demo@omarabulkhair`. Config and cache folders keep the `feature-demo-video` name, so your voice sample and key stay where they were.
- **The fork's maintainer is the only one testing it.** Qwen voices, recording and compose were not re-tested after the rename and the Arabic change.

## License

MIT. The voice models download at setup under their own licences (Qwen3-TTS: Apache-2.0, parakeet-tdt: CC-BY-4.0).
