# xilo-opus-video

English | [中文](README.md)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.9%2B-blue.svg)](https://www.python.org/)
[![JavaScript](https://img.shields.io/badge/JavaScript-Canvas%20%7C%20WebGL-f7df1e.svg)](skills/xilo-opus-video/templates/vertical-explainer)
[![Claude Code Skill](https://img.shields.io/badge/Claude%20Code-Skill-d97757.svg)](skills/xilo-opus-video/SKILL.md)

> Make videos with code: Claude proposes three plans with previews, you pick one, then it builds the video.
>
> Canvas / HTML·CSS·SVG / WebGL shaders / Three.js / Remotion / HyperFrames / Manim | code-synthesized music | agent skill

> This repository is a fork of [Kianzzz/xilo-opus-video](https://github.com/Kianzzz/xilo-opus-video). The original workflow is kept intact; on top of it, this fork adds a full track for **turning a written script into a narrated vertical short video with subtitles and a cover**. See "What this fork adds" below.

---

## What it is

The model does not render video by itself. The pixel animations, UI motion pieces and history shorts you see online are web pages that draw frame t on request, captured frame by frame and encoded into an MP4.

Asking for a whole video in one sentence is a lottery: you cannot see how it was planned, and a good result does not repeat on the next topic. This skill splits the job into steps you can confirm and reuse:

1. **Ask what to make**: an idea, an article, a product URL, a codebase, or a reference video
2. **Three plans**: style, tech stack, shot-by-shot summary, sound and render time, deliberately different from each other
3. **A preview for each plan**: one key frame, or a 2–4 second sample
4. **Build after you confirm**: the chosen plan is written into `brief.md` (a reusable structured prompt), then built shot by shot with a still check after each shot, and rendered to MP4

---

## What this fork adds

The original is great at motion pieces without narration. This fork covers **narrated vertical shorts** (talking-head scripts, explainers). Everything below has been used on real published videos:

- **Voiceover pipeline** (`scripts/voice/`): synthesize the whole narration in one pass (Gemini TTS, or free edge-tts), sped up to 1.25x by default; get word timestamps with Whisper and align them back to your script character by character, so scene changes, on-screen actions and subtitles are all anchored to exact phrases of your script; finally mix music, effects and ducked voice to -14 LUFS.
- **Vertical explainer template** (`templates/vertical-explainer/`): frame 0 is the cover, scenes cross-fade, one-line hard subtitles at the bottom, plus stamps, chat bubbles, cards and a progress row. Swap the script, write a scene table, render.
- **Cover and opening rules** (`references/opening-and-cover.md`): the first frame is the cover, no dates or title cards; a bold claim in the first 5 seconds; the opening sets up the contrast and leaves the answer to the body.
- **Fewer words on screen when there is a voice**: the narration carries sentences, the screen shows keywords that appear when they are spoken.
- **Fact-checking** (`references/fact-check.md` + `scripts/factcheck.py`): prices, quotas, data and deadlines are researched into a claims table (source URL + date + status) and the script is revised to match; before the final render a gate blocks any unverified claim and flags on-screen numbers that are not in the table.
- **Parallel rendering** (`scripts/render_parallel.py`): split the timeline by frame, render ranges in several browsers at once, join losslessly; about 2.8x faster than sequential with the exact same frame count.
- **QA contact sheet** (`scripts/qa_stills.py`): one command renders frame 0 and the fullest state of every scene into a single image.
- **Ready to publish**: besides the MP4, a cover image without subtitles and `out/publish.txt` (title, description, 5 tags, ready to paste); the video and cover are named after the title (`scripts/topic_name.py`), so videos on different topics never share a file name.

Template example (frame 0 cover + the fullest state of each scene):

![Vertical explainer template](examples/vertical-explainer-sheet.png)

---

## Example: one brief, three plan previews

The brief was "a 12-second video introducing this skill, 16:9, no voiceover, code-synthesized music, style up to you". The three plan previews below are all rendered from code.

**Plan A · UI motion** (HTML/CSS/SVG + springs)

![Plan A](examples/plan-a-ui.png)

**Plan B · Pixel cartridge** (Canvas 2D pixel art → WebGL CRT shader)

![Plan B](examples/plan-b-pixel-crt.png)

**Plan C · One line** (SVG line drawing + one continuous tracking shot, frames from a 3-second sample)

![Plan C](examples/plan-c-one-line-sheet.png)

---

## Good for

- Product promos, feature demos, launch-style motion pieces
- Explainers, history recaps, data stories
- Turning a written script into a narrated vertical short with subtitles and a cover (Douyin, WeChat Channels, Xiaohongshu, Reels, Shorts)
- Stylized shorts: pixel art, retro TV, hand-drawn, UI motion
- Rebuilding the structure and techniques of a reference video for your own content
- Building code scenes, big type and effects around **green-screen characters** generated by a video model

Not for:

- Generating photorealistic footage (that needs a video model; this skill handles the code layer)
- Editing existing footage in CapCut, Premiere or After Effects
- Adding subtitles to an existing video

---

## Install

Tell Claude Code:

```text
Install this skill: https://github.com/anneheartrecord/xilo-opus-video
```

Or with the skills CLI:

```bash
npx skills add anneheartrecord/xilo-opus-video
```

Or copy it manually:

```bash
git clone https://github.com/anneheartrecord/xilo-opus-video.git
cp -r xilo-opus-video/skills/xilo-opus-video ~/.claude/skills/
```

### Requirements

- Python 3.9+ with `numpy` and `playwright` (`python3 -m pip install numpy playwright`)
- FFmpeg
- A Chromium browser: `python3 -m playwright install chromium`, or an installed Google Chrome

For narrated videos you also need:

- `scipy` (mixing)
- One voice: the `GEMINI_API_KEY` environment variable (Gemini TTS), or `edge-tts` (free, `uv tool install edge-tts`)
- Whisper for alignment: `mlx-whisper` on Apple Silicon, `faster-whisper` elsewhere; to avoid a global install, run `uv run --with mlx-whisper ...`

On first use the skill runs `scripts/check_env.py` and tells you what is missing.

---

## Usage

Just tell Claude what you want:

```text
Make a 30-second promo for https://example.com, 16:9, code-synthesized music
```

```text
Here is my article (text attached). Turn it into a 60-second vertical explainer with voiceover and subtitles
```

```text
Follow the pacing and transitions of this reference video to make a 15-second clip about our app's new feature (video attached)
```

Everything goes into `opus-video/<project>/` in the current directory:

```
source/      source material, reference frames and analysis
previews/    previews of the three plans
brief.md     the confirmed plan (a reusable structured prompt)
index.html   the page that draws the video
audio/       music, effects, voiceover
qa/          stills and contact sheets for self-checks
out/         <topic>.mp4, <topic>-封面.png, publish.txt
```

---

## What's inside

```
skills/xilo-opus-video/
  SKILL.md                    workflow: brief → three plans → previews → confirm → brief.md → build → review
  references/
    code-stack.md             what each of the eight code types is good at, style ↔ stack mapping
    plan-format.md            how to write the three plans and their previews
    brief-template.md         the seven-block prompt template
    craft-rules.md            frames depend only on time, spring presets, multi-format, sound, scoring, checklist
    opening-and-cover.md      first frame is the cover, 5-second hook, opening timing, cover export (fork)
    voiceover-pipeline.md     narrated videos: command order, scenes.json, known pitfalls (fork)
    fact-check.md             fact-checking: when, how to research, how to revise, gate before rendering (fork)
  scripts/
    check_env.py              checks Python / FFmpeg / Playwright / browser / WebGL
    render.py                 frame-by-frame rendering: stills, full video, segments, motion blur, audio mux
    analyze_video.py          video info, shot changes, contact sheets (reference analysis and final review)
    audio_tools.py            timed sound effects, beat detection, loudness normalization, waveform and spectrum images
    qa_stills.py              frame 0 + fullest state of every scene in one sheet (fork)
    render_parallel.py        parallel rendering: frame ranges, several browsers, lossless join, frame-count check (fork)
    factcheck.py              claims table: extract facts from the script, gate before the final render (fork)
    topic_name.py             file name from the title, so different topics never collide (fork)
    voice/                    tts.py synthesize + speed up / align.py word timestamps / timeline.py scenes and subtitles / mix.py mixing (fork)
  templates/
    vertical-explainer/       vertical explainer template: stage.js engine + a minimal example (fork)
```

---

## Credits

- Original author [周行 (Kianzzz)](https://github.com/Kianzzz): the workflow, references and rendering scripts come from his [xilo-opus-video](https://github.com/Kianzzz/xilo-opus-video), which in turn credits [@twoclipping](https://x.com/twoclipping), [@kimmonismus](https://x.com/kimmonismus), [@majidmanzarpour](https://x.com/majidmanzarpour), [@prasenx](https://x.com/prasenx), [@aiwarts](https://x.com/aiwarts), [@Gorden_Sun](https://x.com/Gorden_Sun) and [@AbaneChan](https://x.com/AbaneChan) for publishing their work and methods.

## License

MIT, keeping the original copyright notice. See [LICENSE](LICENSE).
