# Paathshala — Backend

FastAPI backend for Paathshala. Orchestrates the multi-agent content generation pipeline, handles Google OAuth, and manages lesson plan data.

See the [root README](../README.md) for the full system overview (frontend, agent pipeline, architecture).

## Features

- **7-agent content generation pipeline** (via [Google ADK](https://google.github.io/adk-docs/)):
  - Prompt Parser — extracts topic, grade levels, and content types from the request
  - Cultural Reference Agent — live Google Search + geolocation to surface locally relevant references
  - Grade Mapper — infers grade levels when unspecified
  - Syllabus Agent — searches for real curriculum points (CBSE/ICSE/NCERT/state boards)
  - Enricher Agent — builds a content-type-specific, culturally-aware generation prompt
  - Content Generator — produces the final structured content
  - Story Breaker — splits generated stories into narrated, image-prompted visual segments
- **Google OAuth login** via Authlib, with session-based auth
- **Geolocation lookup** — IP-based geolocation + Google Maps geocoding to localize content
- **Async task dispatch** — content generation jobs run via Celery rather than blocking requests
- **Lesson plan management** — CRUD endpoints for lesson plans, including logic to push lessons to the next working day and reschedule around marked holidays
- **ElevenLabs integration** for voice narration output
- **PostgreSQL** (Supabase-hosted) for persistent storage

## Tech stack

- FastAPI + Uvicorn
- Google ADK (`LlmAgent`, Gemini 2.0/2.5 Flash models)
- Celery — async task queue
- Authlib — Google OAuth
- psycopg2 — PostgreSQL client
- ElevenLabs SDK — voice synthesis

## Project structure

```
main.py       # FastAPI app, routes: auth, lesson plans, holiday rescheduling, generation endpoints
agents.py     # All 7 agent definitions with Pydantic output schemas and instructions
flow.py       # Agent runner, geolocation lookup, ADK session management
task.py       # Async task dispatch: topic generation, content-type generation, full generation flow
req.txt       # Python dependencies
```

## Setup

```bash
pip install -r req.txt
```

Create a `.env` file with:

```
SECRET_KEY=your_flask_session_secret
GOOGLE_CLIENT_ID=your_google_oauth_client_id
GOOGLE_CLIENT_SECRET=your_google_oauth_client_secret
GOOGLE_API_KEY=your_google_ai_api_key
MAPS_API=your_google_maps_api_key
ELEVENLABS_API_KEY=your_elevenlabs_api_key
SUPPABASE_PWD=your_supabase_db_password
```

Run:

```bash
uvicorn main:app --reload
```

Server runs at `http://localhost:8000` by default.

## API overview

- `GET /login`, `GET /auth`, `GET /logout`, `GET /api/me` — Google OAuth flow & session check
- `PUT /lesson-plans/{lesson_id}/push-tomorrow` — move a lesson to the next working day
- `DELETE /lesson-plans/{lesson_id}` — delete a lesson plan
- `PUT /mark-holiday` — mark a date as a holiday and cascade-reschedule affected lessons

(Content generation endpoints are dispatched through `task.py`'s async functions — see `main.py` for the full route list.)

## Status & Limitations

1. **Hardcoded DB connection details** — the Supabase host, database user, and port are hardcoded in `task.py` and `main.py`; only the password is pulled from the environment. Move the rest into `.env` before wider use.
2. **CORS is wide open** (`allow_origins=["*"]`) — fine for local development, should be scoped to specific origins in any deployed version.
3. **No automated tests** currently.
4. **`trial.py`** is a development scratch file, not part of the core app.
