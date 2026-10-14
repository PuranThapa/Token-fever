[README.md](https://github.com/user-attachments/files/33417509/README.md)
# Token Fever — AI-Generated Animated Music Video

Create a beat-synchronized animated music video for **Token Fever**, a song about the everyday life of an AI developer, juggling AI tools, subscriptions, API usage, and a growing token bill.

This repository is designed to be used with an AI coding agent such as **Claude Code**. The agent reads the detailed workflow in [`AI_README.md`](AI_README.md), then helps build and render the music video locally.

## Quick start

1. Clone this repository:

   ```bash
   git clone <YOUR_REPOSITORY_URL>
   cd <REPOSITORY_FOLDER>
   ```

2. Open the project folder.

3. Add your own song audio and lyrics:
   - `assets/token-fever.mp3` (generate using http://aistudio.google.com)
   - `assets/lyrics.txt` (generate lyrics using chatgpt)

   Create the `assets/` folder if it does not exist. Use your own audio and final lyrics.

4. Open **Claude Code** or another capable AI coding agent in the repository folder.

5. Give the agent this instruction:

   > Read `AI_README.md` completely and follow its instructions to create the Token Fever animated music video. Inspect the existing project before changing anything. Clear the contents of `out/` first, because it contains example output, and then create fresh preview and final renders there. Do not delete files outside `out/`.

6. Review the generated preview before asking the agent to render the complete video.

## What the agent should create

- A custom animated music video synchronized to the supplied song
- A storyboard with scene timing
- Source code and rendering scripts
- A preview render and contact sheets for review
- A final MP4 in `out/token-fever.mp4`
- Instructions for reproducing the render

## Requirements

The exact requirements depend on the implementation selected by your AI agent. The intended workflow uses browser-based animation, headless Chrome for rendering, and FFmpeg to combine the frames with the original audio. Install any dependencies the agent identifies, and review commands before running them.

## Project notes

- The music and lyrics are inputs; provide your own files in `assets/`.
- The target output is 1920×1080 at 24 fps, H.264 video with AAC audio, if supported by your machine.
- Render times and resource requirements depend on the complexity of the animation and your hardware.
- Product/model names and any on-screen pricing should be represented accurately. Do not present fictional token bills as real pricing.

See [`AI_README.md`](AI_README.md) for the full production brief and workflow.
