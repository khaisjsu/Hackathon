# Poster AI 🎨

An AI-powered poster generation app. Users describe what they want, and an AI agent designs custom posters for them — generating backgrounds, choosing palettes, composing layouts, and reviewing quality automatically.

Built as a full-stack project with a React frontend and Node.js backend that orchestrates Claude (Anthropic) and DALL·E 3 (OpenAI) into a 5-step agent pipeline.

---

## 📑 Table of Contents

1. [What It Does](#-what-it-does)
2. [Demo Flow](#-demo-flow)
3. [Tech Stack](#-tech-stack)
4. [Project Structure](#-project-structure)
5. [Architecture](#-architecture)
6. [Prerequisites](#-prerequisites)
7. [Installation](#-installation)
8. [Running the App](#-running-the-app)
9. [How It Works (File-by-File)](#-how-it-works-file-by-file)
10. [The Agent Pipeline](#-the-agent-pipeline)
11. [API Endpoints](#-api-endpoints)
12. [Environment Variables](#-environment-variables)
13. [Mock Mode vs Live Mode](#-mock-mode-vs-live-mode)
14. [Customizing the Project](#-customizing-the-project)
15. [Troubleshooting](#-troubleshooting)
16. [Team Roles](#-team-roles)
17. [Roadmap](#-roadmap)
18. [License](#-license)

---

## ✨ What It Does

- **Describe in plain English** — "Summer festival poster, vibrant, youth audience"
- **Pick a style** — Bold, Minimal, Playful, Corporate, or Retro
- **Choose a size** — A4, Square, Story (9:16), Landscape (16:9)
- **Watch the agent work** — Live progress UI shows each step as it happens
- **Get 3 unique variants** — Each with its own palette and AI-generated background
- **Download as PNG** — High-resolution, ready to print or share

---

## 🎬 Demo Flow

1. User opens `http://localhost:5173`
2. Fills out the brief form (prompt, headline, style, size)
3. Clicks **Generate Posters**
4. The agent pipeline runs (~10–20 seconds):
   - ✓ Understanding your brief
   - ✓ Choosing color palette
   - ✓ Generating background visuals
   - ✓ Composing layout
   - ✓ Reviewing quality
5. Three poster variants appear in a gallery
6. User downloads their favorite as PNG

---

## 🛠 Tech Stack

### Frontend
- **React 18** — UI framework
- **Vite** — fast dev server and build tool
- **Tailwind CSS** — utility-first styling
- **Framer Motion** — animations and transitions
- **Lucide React** — icon set

### Backend
- **Node.js 18+** — runtime
- **Express** — HTTP server
- **Sharp** — image composition (background + text overlay → final PNG)
- **Server-Sent Events (SSE)** — live progress streaming

### AI Services
- **Anthropic Claude** — brief expansion + design critique (the "brain")
- **OpenAI DALL·E 3** — background image generation (the "artist")

---

## 📁 Project Structure

```
poster-ai-project/
│
├── README.md                          ← you are here
├── .vscode/
│   └── tasks.json                     ← run both servers from VS Code
│
├── poster-ai/                         ← FRONTEND (port 5173)
│   ├── index.html
│   ├── package.json
│   ├── vite.config.js
│   ├── tailwind.config.js
│   ├── postcss.config.js
│   ├── .env.example
│   └── src/
│       ├── main.jsx                   ← React entry point
│       ├── App.jsx                    ← state machine + backend connection
│       ├── index.css                  ← Tailwind + global styles
│       ├── components/
│       │   ├── InputForm.jsx          ← brief input screen
│       │   ├── AgentProgress.jsx      ← live agent progress UI
│       │   ├── ResultsGallery.jsx     ← variants grid + download
│       │   └── PosterPreview.jsx      ← (legacy mock renderer)
│       └── data/
│           └── templates.js           ← style presets, sizes, options
│
└── poster-ai-backend/                 ← BACKEND (port 3001)
    ├── package.json
    ├── .env.example
    ├── README.md
    ├── output/                        ← generated PNGs land here
    └── src/
        ├── server.js                  ← Express app entry
        ├── routes/
        │   └── generate.js            ← API endpoints (sync + SSE)
        ├── agent/
        │   └── orchestrator.js        ← runs the 5-step pipeline
        └── services/
            ├── llm.js                 ← Claude calls
            ├── imageGen.js            ← DALL·E 3 calls
            └── composer.js            ← Sharp text overlay → PNG
```

---

## 🏗 Architecture

```
   User in Browser
         │
         ▼
   ┌──────────────────────────────────┐
   │  Frontend :5173 (React + Vite)   │
   │  • InputForm                     │
   │  • AgentProgress (SSE consumer)  │
   │  • ResultsGallery                │
   └────────┬─────────────────────────┘
            │ POST /api/generate/stream
            │ (Server-Sent Events)
            ▼
   ┌──────────────────────────────────┐
   │  Backend :3001 (Express)         │
   │                                  │
   │  ┌────────────────────────────┐  │
   │  │  Agent Orchestrator        │  │
   │  │   1. Expand brief    ──────┼──┼──► Anthropic Claude API
   │  │   2. Pick palette          │  │
   │  │   3. Generate images ──────┼──┼──► OpenAI DALL·E 3 API
   │  │   4. Compose posters       │  │
   │  │   5. Critique        ──────┼──┼──► Anthropic Claude API
   │  └──────────┬─────────────────┘  │
   │             ▼                    │
   │       saves PNGs to              │
   │       output/*.png               │
   │             │                    │
   │       served via                 │
   │       GET /output/:filename      │
   └─────────────┬────────────────────┘
                 │
                 ▼
           Browser displays PNGs
```

---

## ✅ Prerequisites

Before you start, make sure you have:

- **Node.js 18 or newer** — check with `node --version`
- **npm** (comes with Node) — check with `npm --version`
- **VS Code** (or any code editor)
- **A modern browser** — Chrome, Firefox, Safari, or Edge

Optional (only for live mode, not needed for development):
- An [Anthropic API key](https://console.anthropic.com)
- An [OpenAI API key](https://platform.openai.com)

---

## 📥 Installation

### 1. Get the project

Unzip `poster-ai-project.zip` and open the folder in VS Code:

```bash
cd poster-ai-project
code .
```

### 2. Install backend dependencies

Open a terminal in VS Code (`` Ctrl+` ``):

```bash
cd poster-ai-backend
npm install
cp .env.example .env
```

The default `.env` is set up with `MOCK_MODE=true` — you don't need any API keys to start.

### 3. Install frontend dependencies

Open a **second terminal** (click the `+` icon in the terminal panel):

```bash
cd poster-ai
npm install
cp .env.example .env
```

The default `.env` points to `http://localhost:3001` for the backend.

---

## 🚀 Running the App

You always need **two terminals running side by side.**

### Terminal 1 — Backend

```bash
cd poster-ai-backend
npm run dev
```

You should see:
```
🎨 Poster AI backend running on http://localhost:3001
   Mock mode: ON
```

### Terminal 2 — Frontend

```bash
cd poster-ai
npm run dev
```

You should see:
```
VITE ready in 400ms
➜ Local: http://localhost:5173/
```

### Open the app

Go to **http://localhost:5173** in your browser. You're live!

### VS Code Shortcut

Instead of two terminals, press `Ctrl+Shift+P` → **"Run Task"** → **"Start All"**. Both servers launch in their own panels (config is in `.vscode/tasks.json`).

---

## 📂 How It Works (File-by-File)

### Frontend Files

#### `src/App.jsx`
The main state machine. Tracks which screen the user sees (`input` → `generating` → `results`) and handles the connection to the backend. When the user submits a brief, it opens an SSE connection to `/api/generate/stream` and streams progress updates into the UI.

#### `src/components/InputForm.jsx`
The brief form — prompt textarea, headline/subtext inputs, style chips, purpose chips, size selector. Validates input and calls `onGenerate(brief)` when the user clicks the button.

#### `src/components/AgentProgress.jsx`
The live progress UI. Receives `currentStep` from `App.jsx` (driven by real SSE events from the backend) and animates checkmarks as each step completes.

#### `src/components/ResultsGallery.jsx`
Renders the 3 generated posters as cards. Each card shows the PNG (loaded from the backend), a quality score from the critique step, and a download button.

#### `src/data/templates.js`
Static reference data — style presets, size options, purpose options. Used by the form to populate the chip selectors.

### Backend Files

#### `src/server.js`
Express app entry. Sets up CORS (so the frontend can talk to it), registers routes, and serves the `output/` folder as static files (so the browser can fetch generated PNGs).

#### `src/routes/generate.js`
Two endpoints:
- `POST /api/generate` — synchronous, returns the result when done
- `POST /api/generate/stream` — SSE streaming, sends live step updates

Both validate the incoming brief and pass it to the orchestrator.

#### `src/agent/orchestrator.js`
The heart of the agent. Runs 5 steps in sequence (with parallelization where it makes sense), calling each service in turn and emitting `onStep` events for the SSE stream.

#### `src/services/llm.js`
Wraps Anthropic Claude calls:
- `expandBrief(brief)` — turns the user's prompt into structured design specs (palettes, image prompts, layout)
- `critiquePoster(variant, brief)` — scores the result for quality

Falls back to deterministic mock data when `MOCK_MODE=true`.

#### `src/services/imageGen.js`
Wraps OpenAI DALL·E 3 image generation. Takes a text prompt and returns an image URL. In mock mode, uses [picsum.photos](https://picsum.photos) for free placeholder images.

#### `src/services/composer.js`
Where the magic visually comes together. Takes the AI-generated background, downloads it, builds an SVG with the headline and subtext text, composites the SVG on top of the background using **Sharp**, and saves the final PNG to `output/`.

---

## 🔄 The Agent Pipeline

When you click "Generate", here's exactly what happens:

| Step | Service | What Happens | Outputs |
|------|---------|--------------|---------|
| 1 | `llm.js` | Claude reads the brief and expands it into design specs | Palettes, image prompts, layout hints |
| 2 | (in orchestrator) | Palettes selected from Claude's expansion | 3 color palettes |
| 3 | `imageGen.js` | DALL·E generates 3 backgrounds in parallel | 3 image URLs |
| 4 | `composer.js` | Sharp + SVG overlays text on each background | 3 PNG files in `output/` |
| 5 | `llm.js` | Claude scores each poster for quality | Critique scores |

The whole thing runs in 10–20 seconds (mostly waiting on DALL·E).

---

## 🌐 API Endpoints

### `GET /health`
Returns server status.

```json
{ "status": "ok", "mockMode": true }
```

### `POST /api/generate`
Synchronous generation. Returns final result when done.

**Request body:**
```json
{
  "prompt": "Summer music festival, vibrant",
  "headline": "SUMMER FEST",
  "subtext": "July 15 · Golden Gate Park",
  "style": "bold",
  "purpose": "event",
  "size": "a4"
}
```

**Response:**
```json
{
  "jobId": "uuid-here",
  "brief": { ... },
  "expanded": { "interpretation": "...", "palettes": [...], ... },
  "variants": [
    {
      "index": 0,
      "url": "/output/uuid-0.png",
      "palette": { "bg": "#...", "accent": "#...", "text": "#..." },
      "critique": { "scores": {...}, "overall": 8, "notes": "..." }
    }
  ]
}
```

### `POST /api/generate/stream`
Same input, but streams progress via Server-Sent Events.

**Events:**
- `step` — `{ id, status: 'running'|'done', label, data? }`
- `complete` — final result (same shape as `/generate`)
- `error` — `{ message }`

### `GET /output/:filename`
Serves the generated poster PNGs.

---

## 🔐 Environment Variables

### Backend (`poster-ai-backend/.env`)

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `PORT` | No | `3001` | Backend server port |
| `FRONTEND_URL` | No | `http://localhost:5173` | Allowed CORS origin |
| `MOCK_MODE` | No | `false` | Skip real API calls — use mock data |
| `ANTHROPIC_API_KEY` | Only in live mode | — | Get from console.anthropic.com |
| `OPENAI_API_KEY` | Only in live mode | — | Get from platform.openai.com |

### Frontend (`poster-ai/.env`)

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `VITE_API_URL` | No | `http://localhost:3001` | Backend URL the frontend calls |

---

## 🎭 Mock Mode vs Live Mode

### Mock Mode (default)

Set `MOCK_MODE=true` in the backend's `.env`. The backend will:
- Use hardcoded palette/prompt expansions
- Pull placeholder images from picsum.photos
- Skip real LLM critique calls
- Still produce real composed posters (just with stock backgrounds)

**Use this for development.** No API costs, full pipeline still runs.

### Live Mode

Set `MOCK_MODE=false` and add real API keys. The backend will:
- Call Claude for actual brief expansion and critique
- Call DALL·E 3 to generate unique backgrounds
- Produce truly custom posters

**Cost estimate per generation:**
- DALL·E 3 standard images (3 variants): ~$0.12
- Claude calls (brief expansion + 3 critiques): ~$0.02
- **Total: ~$0.14 per poster generation**

---

## 🎨 Customizing the Project

### Add a new style preset

1. Open `poster-ai-backend/src/services/llm.js`
2. Add a new key to the `palettes` object in `mockExpansion()`
3. Open `poster-ai/src/data/templates.js`
4. Add the new style to `stylePresets` array

### Change the agent steps

Edit the `steps` array in `poster-ai/src/components/AgentProgress.jsx` AND the corresponding `onStep()` calls in `poster-ai-backend/src/agent/orchestrator.js`. Keep them in sync.

### Improve the poster typography

Edit `buildTextOverlay()` in `poster-ai-backend/src/services/composer.js`. The function builds an SVG, so you can adjust fonts, positioning, decorative elements, etc.

### Add new poster sizes

1. Add to `DIMENSIONS` in `poster-ai-backend/src/services/composer.js`
2. Add to `SIZE_MAP` in `poster-ai-backend/src/services/imageGen.js`
3. Add to `sizeOptions` in `poster-ai/src/data/templates.js`

### Use a different image provider

Replace the body of `generateBackground()` in `services/imageGen.js`. As long as it returns `{ url, size }`, the rest of the pipeline doesn't care which provider you use (Stable Diffusion, Flux, Imagen, etc.).

---

## 🐛 Troubleshooting

### "Failed to fetch" or CORS errors
The backend isn't running, or it's on the wrong port. Check Terminal 1.

### Posters don't show up in the gallery
- Open browser DevTools → Network tab → look for the `/api/generate/stream` request
- Check Terminal 1 for backend errors
- Look in `poster-ai-backend/output/` — are PNGs being created?

### "ANTHROPIC_API_KEY is not set" error
You're not in mock mode. Either:
- Set `MOCK_MODE=true` in `poster-ai-backend/.env`, OR
- Add your real API keys

### Agent progress stays stuck
The browser is buffering the SSE stream. Try a different browser or disable browser extensions.

### `npm install` fails on Sharp
Sharp requires native binaries. On Linux, you may need build tools:
```bash
sudo apt-get install -y build-essential
```
On macOS, Xcode Command Line Tools usually cover it.

### Port already in use
Something else is on port 3001 or 5173. Either stop that process, or change the port in `.env`.

---

## 👥 Team Roles

This project was designed for a 3-person team:

| Role | Responsibility | Files They Own |
|------|----------------|----------------|
| **PM / Prompt Lead** | Coordination, prompt engineering, QA | `services/llm.js` prompts, agent step labels |
| **Backend / AI Engineer** | Server, agent logic, API integrations | All of `poster-ai-backend/src/` |
| **Frontend / Design Engineer** | UI, UX, animations | All of `poster-ai/src/` |

---

## 🗺 Roadmap

### Phase 1 — MVP (current)
- [x] Frontend with input form, agent progress, results gallery
- [x] Backend with 5-step agent pipeline
- [x] Mock mode for development
- [x] SSE streaming for live progress
- [x] PNG export

### Phase 2 — Production-Ready
- [ ] User accounts and history
- [ ] Cloud storage for generated posters (S3/R2)
- [ ] Job queue (BullMQ + Redis) for parallel users
- [ ] Rate limiting per user
- [ ] Cost tracking dashboard
- [ ] Cache LLM responses for identical briefs

### Phase 3 — Advanced Features
- [ ] In-browser editor (edit text, swap colors, reposition elements)
- [ ] Brand kit upload (logo, brand colors → injected into generations)
- [ ] More export formats (PDF, SVG)
- [ ] Variant remix ("more like this one")
- [ ] Multimodal critique (Claude actually looks at the image)
- [ ] Optional: expose the generator as an MCP server for AI assistants

---

## 📄 License

MIT — do whatever you want with it.

---

## 🙏 Credits

- AI brain: [Anthropic Claude](https://anthropic.com)
- AI artist: [OpenAI DALL·E 3](https://openai.com)
- Image composition: [Sharp](https://sharp.pixelplumbing.com)
- UI framework: [React](https://react.dev) + [Vite](https://vitejs.dev)
- Styling: [Tailwind CSS](https://tailwindcss.com)
- Animations: [Framer Motion](https://www.framer.com/motion/)

---

**Built by [Your Team] — happy posting! 🎉**
