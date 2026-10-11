# Hacklanta 26: Song Pipelines & Pitch Battle

This workspace contains the Musiggle karaoke player, its FastAPI server, and an offline song analysis pipeline. The host screen lives in the root `index.html` and is copied to `pitch_battle/static/index.html` for the FastAPI app. Both launch paths use ngrok for secure phone pairing.

## Current Multiplayer Arena

From the repository root:

```bash
cd pitch_battle
python app.py
```

Open [http://localhost:8000/](http://localhost:8000/) on the host computer. The server starts or reuses an ngrok tunnel on port 8000 and displays a QR code / phone link for players. ngrok must be installed and authenticated. If ngrok is unavailable, phones can connect over the same Wi-Fi using the local address shown in the app.

At the end of each round, the scoreboard can show a two-sentence Gemini Coach recap explaining the winner and giving the runner-up a specific tip. The call runs on the server with Gemini 3.8 Flash at low thinking effort. Locally, put `GEMINI_API_KEY` in the ignored project-root `.env` file or set it in the server process environment; the key is never placed in browser code. If it is unset, the game still runs and the scoreboard explains that Gemini is not configured.

```bash
export GEMINI_API_KEY="your-rotated-key"
cd pitch_battle
python app.py
```

For PowerShell, set `$env:GEMINI_API_KEY = "your-rotated-key"` before starting `app.py` or `run_server.bat`.

## Public Hosting on Render

This project is configured for Render as a Python web service. Push the project to a Git provider, create a Render Blueprint from the repository, and Render will read [`render.yaml`](render.yaml). Enter `GEMINI_API_KEY` in Render when prompted, or add it in the service's Environment settings. Do not upload or commit `.env`.

The service runs `pitch_battle/app.py`, serves the browser app and song assets over Render HTTPS, and uses Render's public hostname for phone links. Render supports the WebSockets used by the alternate room engine. The Blueprint keeps one service instance because room state currently lives in process memory; increasing the instance count can split one battle across servers. The free plan sleeps when idle, so the first visit after a quiet period can take about a minute to wake. Active rounds can be interrupted by deploys or service restarts because room state is not persisted.

The current host page pairs phones directly through PeerJS/WebRTC, so microphone audio goes from each phone to the host browser. The server provides HTTPS, signaling access, songs, and Gemini recaps. For a production service with multiple simultaneous rooms or replicas, room coordination needs shared state and WebSocket message routing.

The current host screen uses PeerJS for phone microphone pairing and the root `songs/` catalog for tracks. `pitch_battle/app.py` provides the ngrok tunnel and serves that song catalog to the deployed page. The existing WebSocket room engine and its separate mobile page remain available through the FastAPI API.

## Standalone WebRTC Player

The repository root also contains the standalone WebRTC player. Run it with:

```bash
python run_pitchbattle.py
```

It serves the root `index.html` and `style.css`, uses PeerJS/WebRTC for phone-to-host microphone audio, and publishes the page through ngrok. The same page is copied into `pitch_battle/static/index.html` so FastAPI displays the same design.

## Offline Song Analysis Pipeline

`song_pipeline_v5/` extracts pitch and note data from reference audio, can align transcribed lyrics with pitch, and compares a recorded or saved cover for pitch, timing, lyric, and note accuracy. Its `compare.py` command is the CLI entry point. See [`song_pipeline_v5/README.md`](song_pipeline_v5/README.md) for setup and examples.

## Other Files

- `songs/`: Shared song catalog and audio/reference assets used by the standalone player.
- `requirements.txt`: Runtime dependencies for the FastAPI arena and audio pipeline.
- `pitch_battle/run_server.bat`: Windows launcher for the current arena.
