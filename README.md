# Paathshala (पाठशाला)

A multi-agent AI system that generates culturally-localized, multi-language lesson content for teachers — stories, worksheets, diagrams, flashcards, and activities — tailored to grade level, subject, and the teacher's own region.

Built as an extended project for the **Google Agentic Hackathon**, using [Google ADK](https://google.github.io/adk-docs/) for agent orchestration.


## What it does

A teacher describes what they need — by typing or by voice, in one of 12 supported languages/dialects — and Paathshala runs it through a pipeline of specialized agents to produce ready-to-use classroom material:

1. **Prompt Parser Agent** — extracts the topic, grade levels, and requested content types (story, worksheet, diagram, activity, flashcards, "best out of waste") from the teacher's request
2. **Cultural Reference Agent** — uses live Google Search, combined with the teacher's geolocation (via IP → Google Maps geocoding), to pull in locally relevant festivals, folk tales, customs, and landmarks
3. **Grade Mapper Agent** — infers appropriate grade levels when the teacher doesn't specify one
4. **Syllabus Agent** — searches for real curriculum/syllabus points (CBSE, ICSE, NCERT, state boards) to keep generated content aligned with what's actually taught
5. **Enricher Agent** — builds a content-type-specific, grade-appropriate, culturally-woven prompt
6. **Content Generator Agent** — produces the final content (story, worksheet, diagram, etc.) in structured JSON
7. **Story Breaker Agent** — for stories, breaks the narrative into synchronized narration + image-prompt segments to generate an interactive visual story

On top of the core pipeline:

- **Weekly lesson planner** — schedule generated content across a week, with logic to auto-reshuffle lessons around holidays and weekends
- **Voice input** — speech-to-text in English, Hindi, Marathi, Bengali, Gujarati, Kannada, Malayalam, Punjabi, Tamil, Telugu, and Urdu
- **Voice output** — ElevenLabs integration for audio narration
- **PDF export & printing** of generated content
- **Offline-capable caching** — generated content and weekly plans are cached locally in IndexedDB
- **Google OAuth login**
- **Async generation** — content generation jobs are dispatched via Celery rather than blocking the request

## Architecture

```
Paathshala/
  frontend/     # React app — search/voice UI, content viewer, weekly planner, PDF export
  backend/      # FastAPI app — agent orchestration, OAuth, Celery tasks, Postgres access
```

### Frontend (`frontend/`)

- **Stack:** React 19, React Router, `idb` (IndexedDB), `react-speech-recognition`, `jsPDF` + `html2canvas`, Framer Motion, Mermaid (for diagram rendering)
- **Key components:**
  - `SearchHome.js` — voice/text input with language selection
  - `ContentGenerator.js` — displays generated content, handles PDF/image export and interactive story generation
  - `WeeklyPlanner.js` — lesson scheduling UI
  - `Resources.js` — saved content library
  - `storage.js` — IndexedDB layer for offline caching

### Backend (`backend/`)

- **Stack:** FastAPI, Google ADK (`LlmAgent`), Celery, PostgreSQL (via Supabase), Authlib (Google OAuth), ElevenLabs SDK
- **Key files:**
  - `agents.py` — all 7 agent definitions with their Pydantic output schemas and instructions
  - `flow.py` — agent runner, geolocation lookup, session management
  - `task.py` — async task dispatch (topic generation, content-type generation, full generation flow)
  - `main.py` — FastAPI routes: auth, lesson plan CRUD, holiday rescheduling, content generation endpoints

## Setup

### Backend

```bash
cd backend
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

### Frontend

```bash
cd frontend
npm install
npm start
```

By default the frontend expects the backend at `http://localhost:8000` — update the API base URL in the frontend source if deploying separately.

## Status & Limitations

1. **Hackathon-origin project, since extended** — the core pipeline works end-to-end, but it hasn't been hardened for production classroom use.
2. **Hardcoded DB connection details** — the Supabase host/user/port are hardcoded in `task.py` and `main.py` (only the password comes from an environment variable). Fine for a personal project, but worth moving fully into env config before wider use.
3. **CORS is wide open** (`allow_origins=["*"]`) — intentional for hackathon/demo convenience, should be scoped to specific origins before production.
4. **Frontend API URL is hardcoded** to `localhost:8000` in places — needs to be made configurable (env variable) for deployment flexibility.
5. **No automated tests** beyond the default Create React App test scaffold.
6. **Multi-agent latency** — a single content request can chain 3–5 agent calls plus live web searches, so generation isn't instant; this is mitigated by Celery's async dispatch rather than blocking the UI.

## Credits

Originally built for the Google Agentic Hackathon. Voice: ElevenLabs. Agent orchestration: Google ADK. Diagrams: Mermaid.
