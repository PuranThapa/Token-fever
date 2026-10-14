# TOKEN FEVER — animated music video

A beat-synchronised, hand-painted 2D music video for the original song *Token Fever*,
built with **p5.js + p5.brush** (painted assets) and Canvas 2D (per-frame ink, characters,
typography), rendered deterministically in **headless Chrome** and muxed with the original
audio by **FFmpeg**. (The original creative brief is preserved in [`BRIEF.md`](BRIEF.md).)

| Deliverable | Location |
|---|---|
| Final video (1920×1080, 24 fps, H.264 High + AAC 320 kbps, 174.25 s, 4182 frames) | `out/token-fever.mp4` |
| Preview clip (pre-chorus → chorus 1 → verse 2, 30–62 s) | `out/preview_30-62s.mp4` |
| Contact sheets (one frame every 1.5 s, exact timestamps) | `out/contact_sheets/sheet_*.jpg` |
| Sync strips (24 frames/s over key hits, with waveform) | `out/contact_sheets/sync_*.jpg` |
| Character sheet | `out/contact_sheets/character_sheet_mira.png` |
| Storyboard with measured timestamps | [`STORYBOARD.md`](STORYBOARD.md) |
| Source | `src/`, `tools/` |

## How the timing works (nothing is guessed)

1. `tools/analyze_audio.py` (librosa) measures from `assets/token-fever.mp3`:
   duration **174.21 s**, tempo **120.07 BPM** (constant grid fitted to tracked beats),
   downbeat phase (chosen by the strongest drum-entry jumps), kick / snare / hat onsets via
   HPSS + band flux, per-frame envelopes (RMS, low, mid, high, vocal) and structural novelty.
   → `src/data/analysis.json`
2. `tools/transcribe.py` (faster-whisper, word timestamps) gives vocal timing;
   `tools/align_lyrics.py` aligns it to the real lyric lines in `assets/layric.txt`.
   → `src/data/lyrics.json`. The final chorus is sung in fewer bars than written, so those
   lines are pinned to measured phrase boundaries and the uncertain stretch (144.0–147.6 s)
   is shown without captions.
3. Scenes read only that data: section cuts on measured bar times (`TF.barT(n)`), letters and
   gags on sung word times (`TF.wordT(line, 'word')`), particle bursts / camera punches /
   pulses on onsets (`TF.kick(t)`, `TF.beatPulse(t)`), lip-sync from the vocal envelope gated
   by word spans (`TF.sing(t)`).

Key measured anchors: drums enter 16.107 s · chorus 1 drop 40.092 s · rap 72.072 s ·
chorus 2 104.052 s · bridge 120.042 s · "Subscribe" click 139.03 s · drums stop 152.022 s ·
"Cha-ching!" 161.96 s (drawer opens on the 162.000 s frame, kick at 162.03 s).

## Setup

Requirements: Node ≥ 18, Python ≥ 3.10, FFmpeg, and a Chrome / chrome-headless-shell binary.

```bash
npm install                       # p5@2.2.3, p5.brush@2.2.3, puppeteer-core
python3 -m venv .venv && .venv/bin/pip install -r requirements.txt   # only needed to re-analyse audio
# Chrome: tools/chrome.js auto-detects ~/.cache/puppeteer/{chrome-headless-shell,chrome}/…,
# or set CHROME=/path/to/chrome. To install one:
npx @puppeteer/browsers install chrome-headless-shell@stable
```

Fonts (Bangers, Caveat / Caveat Brush, Space Mono, Rubik Mono One — SIL OFL) are bundled in `src/fonts/`.

## Render

```bash
# (optional) re-run the audio analysis + lyric alignment
mkdir -p build/audio
ffmpeg -i assets/token-fever.mp3 -ac 1 -ar 22050 build/audio/mono22k.wav
ffmpeg -i assets/token-fever.mp3 -ac 1 -ar 16000 build/audio/mono16k.wav
.venv/bin/python tools/analyze_audio.py
.venv/bin/python tools/transcribe.py small.en && .venv/bin/python tools/align_lyrics.py

# full video (≈3–4 min on an 8-core M1 Pro with 6 workers)
node tools/render.js --workers 6 --crf 20 --preset slow      # → out/token-fever.mp4

# a preview segment
node tools/render.js --start 30 --end 62 --out out/preview.mp4 --preset fast

# individual stills for inspection
node tools/stills.js build/stills 40.2 72.9 139.05 162.0
tools/montage.sh sheet.jpg 3 build/stills/*.jpg
```

Live preview in a browser: `node tools/serve.js 8080`, then open
`http://127.0.0.1:8080/src/index.html?t=40.3` (single frame) or `?play=1&t=30` (plays the song,
click to start/pause; drawing is real-time-ish, the renderer is the source of truth).

`render.js` launches N headless Chrome instances, each rendering a contiguous frame range via
`window.renderFrame(frame / 24)` and piping JPEG frames into its own FFmpeg H.264 segment; the
segments are concatenated losslessly and muxed with the untouched original MP3 → AAC. The video
(4182 frames = 174.25 s) is never shorter than the audio, so nothing is clipped.

## Code map

```
src/index.html          loads p5, p5.brush and the modules below
src/js/core.js          seeded RNG/noise, easing, timeline + beat/onset/lyric helpers, palette
src/js/paint.js         p5.brush painting (WEBGL, job queue flushed per draw): paper, watercolor
                        granulation, wash sprites, brush strokes, splats, room/desk/night/digital/hot backgrounds
src/js/ink.js           per-frame hand-drawn primitives: 12 fps line boil, watercolor fills with
                        pooled edges, halftone shading, IK limbs, glows
src/js/mira.js          the character rig (expressions, lip-sync mouths, IK arms, legs, lighting layer)
src/js/props.js         laptop, mugs, windows, terminals, cards, receipts, coins, clock, meter, register…
src/js/motifs.js        original AI-tool motifs (names only, no logos)
src/js/type.js          kinetic typography, word-synced captions, counters
src/js/fx.js            analytic particles, rays, wipes/iris transitions, camera, paper/grain post
src/js/director.js      scene table + transitions on measured bar times
src/js/scenes/*.js      intro, verse1, prechorus, chorus (1, 2, final), verse2, rap, bridge, outro
tools/                  analysis, alignment, server, renderer, stills, montage
```

Determinism: every visual is a pure function of `t`; painted assets are seeded (identical in
every worker); particles are closed-form; line boil is keyed to `floor(t·12)`.

## Notes on content

- AI products appear as original hand-drawn motifs with their names as text — no logos.
  "Claude Fable" and "ChatGPT Astra" are shown exactly as sung, as fictional "new model" cards.
- No real prices: subscription cards carry tier names only, receipts total `$$$`, tokens are
  counted in tokens. The only dollar figure is Mira's own escalating bill on her phone,
  labelled "(fictional)".
- `build/` holds intermediate files (decoded audio, transcripts, test stills, render segments)
  and can be deleted.
