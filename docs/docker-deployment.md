# Docker Deployment Documentation — OpenNotetaker

## 1. Container Architecture Overview

OpenNotetaker consists of two primary containerized services defined in `docker-compose.yml`:

```
┌─────────────────────────────────────────────────────────┐
│                    Host Machine / EC2                   │
│                                                         │
│  ┌───────────────────────┐   ┌───────────────────────┐  │
│  │   open-notetaker      │   │ open-notetaker-worker │  │
│  │  (Node.js Web App)    │   │  (Bot / Browser / STT)│  │
│  │                       │   │                       │  │
│  │ Port 5173 (127.0.0.1) │◄──│ Talks over docker-net │  │
│  │ Volume:               │   │ Volume:               │  │
│  │   open-notetaker-data │   │   bot-profile (ro)    │  │
│  └───────────────────────┘   └───────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

### Services Breakdown

1. **`open-notetaker` (Web Application)**
   - **Image:** `node:22-bookworm-slim`
   - **Port:** `127.0.0.1:5173:5173` (kept private, exposed via reverse proxy)
   - **Role:** Handles UI, user auth, meeting store, transcript processing, and job dispatching.
   - **Volume:** `open-notetaker-data` mounted to `/app/data`.

2. **`open-notetaker-worker` (Recording Worker)**
   - **Image:** `node:22-bookworm-slim` (with Chromium, FFmpeg, Xvfb, PulseAudio)
   - **Command:** `npm run bot:worker`
   - **Role:** Claims recording jobs, launches headless/rawcdp Chromium into Google Meet, streams audio/video back to main server.
   - **Volume:** `open-notetaker-bot-profile` mounted read-only to `/seed/bot-profile`.

---

## 2. Environment Variables & Secret Management

Secrets must be supplied via `.env` file (never committed to Git).

### Required Environment Variables
- `RUNNER_TOKEN`: Shared secret key between web app and workers.
- `DEEPGRAM_API_KEY`: Required for live Speech-to-Text.
- `GEMINI_API_KEY` (or `OPENAI_API_KEY`): Required for meeting summary & action item generation.
- `OPENNOTETAKER_BASE_URL`: Public HTTPS URL of the application.

---

## 3. Deployment Commands

```bash
# Prepare environment file
cp .env.example .env

# Build container images
docker compose build

# Launch services in detached mode
docker compose up -d

# Scale recording workers (e.g., 3 workers for concurrent recording)
docker compose up -d --scale open-notetaker-worker=3

# View container logs
docker compose logs -f

# Check status
docker compose ps
```

---

## 4. Verification & Data Persistence

- **Data Persistence:** Persistent meeting notes, transcripts, and media files are stored in `open-notetaker-data` volume (`/app/data`). Data survives container restarts and upgrades.
- **Worker Isolation:** Workers use independent container storage for Chrome profiles (`/app/.bot-profile-local`) so concurrent meetings do not interfere.
