# Paathshala — Frontend

React frontend for Paathshala, a multi-agent lesson-content generator for teachers. Handles the search/voice interface, content display, weekly lesson planner, and PDF/image export.

See the [root README](../README.md) for the full system overview (backend, agent pipeline, architecture).

## Features

- **Voice & text search** — describe what content you need by typing or speaking, in one of 12 supported languages/dialects (English, Hindi, Marathi, Bengali, Gujarati, Kannada, Malayalam, Punjabi, Tamil, Telugu, Urdu)
- **Content viewer** — displays generated stories, worksheets, diagrams, flashcards, and activities per grade/type
- **Interactive visual stories** — turns a generated story into a segmented, narrated visual sequence
- **Weekly planner** — schedule generated lessons across the week
- **PDF & image export** — download generated content as a print-ready PDF or image
- **Offline-capable caching** — saved content and weekly plans persist locally via IndexedDB, so previously generated material is available without a network round-trip
- **Google OAuth session handling**

## Tech stack

- React 19 + React Router
- `idb` — IndexedDB wrapper for local caching
- `react-speech-recognition` — voice input
- `jsPDF` + `html2canvas` — PDF/image export
- Framer Motion — animation
- Mermaid — diagram rendering
- Lucide — icons

## Project structure

```
src/
  App.js                      # Routing, auth check, theme state
  components/
    SearchHome.js               # Voice/text search input, language selector
    ContentGenerator.js           # Content display, PDF/image export, visual story trigger
    WeeklyPlanner.js                # Lesson scheduling UI
    Resources.js                     # Saved content library view
    LandingPage.js                    # Landing/marketing page
    Navbar.js                          # App navigation
    RotatingText.js                     # Rotating language-name animation on landing page
    storage.js                           # IndexedDB schema & CRUD (saved content + weekly plans)
public/                                    # Static assets, manifest
apphosting.yaml                              # Firebase App Hosting config
```

## Setup

```bash
npm install
npm start
```

Runs at `http://localhost:3000` by default and expects the backend running at `http://localhost:8000` (update the API base URL in the source if your backend runs elsewhere).

## Build

```bash
npm run build
```

Outputs a production build to `build/` — this folder is generated and shouldn't be committed (see root `.gitignore`).

## Status

Voice input relies on the Web Speech API via `react-speech-recognition`, which has inconsistent browser support (works best in Chrome-based browsers). API base URLs are currently hardcoded to `localhost` in a few places and should be moved to an environment variable before deploying to a non-local backend.
