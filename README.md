# Internpath — Internship Guidance Agent

An internship discovery and guidance platform built with React, FastAPI, and Python. Students can browse a curated catalog of internship opportunities, get AI-assisted career coaching, track applications, and analyze profile–job fit.

> **Demo mode:** The application ships with a full in-memory demo mode that requires no database or API keys to run locally. All data resets on server restart. Production use requires MongoDB Atlas and optionally an LLM provider key.

---

## Features

- **Opportunity catalog** — Browse source-linked internship listings with filters for role, company, skill, location, and work mode.
- **Profile-based matching** — Enter your degree, year, CGPA, skills, and location preferences to get ranked recommendations.
- **AI assistant** — Bounded chat interface backed by OpenAI, Gemini, or Groq (falls back to deterministic mock if no key is set).
- **Application tracker** — Log and update application status, deadlines, follow-up dates, resume version, and notes per role.
- **Student progress hub** — Track learning milestones, review mock-interview history, and evaluate a public GitHub project.
- **Resume tools** — Upload a PDF or DOCX résumé to extract editable text, validate basic formatting, and generate a clean download.
- **JWT authentication** — Register and log in with email and password; tokens are short-lived and scoped per session.
- **Data sync scripts** — Seed India-focused internships from official employer pages, or pull recent listings from The Muse public API.

---

## Technology stack

| Layer | Technology |
|---|---|
| Frontend | React 18, Vite, lucide-react |
| Backend | FastAPI, Uvicorn, Python 3.11 |
| Database | MongoDB Atlas (production) / in-memory (demo) |
| Auth | JWT (PyJWT), Argon2 password hashing (pwdlib) |
| AI | OpenAI / Gemini / Groq (auto-selected) or deterministic mock |
| Document processing | pypdf, python-docx, reportlab |
| Testing | pytest |

A legacy static frontend (`css/`, `js/`, `*.html` at the project root) is preserved for compatibility but is not actively maintained. The primary interface is the React app in `frontend-react/`.

---

## Project structure

```
internship-guidance-agent/
├── backend/                    # FastAPI application
│   ├── agent_tools/            # LLM tool definitions (search, compare, skill analysis)
│   ├── routes/                 # API route handlers
│   ├── scripts/                # Data seed and sync scripts
│   ├── services/               # Business logic (matching, LLM, resume, safety)
│   ├── tests/                  # pytest test suite (80 tests)
│   ├── utils/                  # JWT and password helpers
│   ├── config.py               # Pydantic settings loaded from .env
│   ├── database.py             # MongoRepository and MemoryRepository
│   ├── main.py                 # FastAPI app factory
│   ├── models.py               # Pydantic request/response models
│   ├── requirements.txt        # Python dependencies
│   └── .env.example            # Environment variable template
├── frontend-react/             # React + Vite application
│   ├── src/
│   │   ├── main.jsx            # Single-file React app (all components)
│   │   └── styles.css          # Application styles
│   ├── index.html
│   ├── package.json
│   └── .env.example            # Frontend environment template
├── css/                        # Legacy static frontend styles
├── js/                         # Legacy static frontend scripts
└── *.html                      # Legacy static frontend pages
```

---

## Prerequisites

- **Python 3.11+** (tested with 3.11.7 via Anaconda)
- **Node.js 18+** and **npm 9+** (tested with Node 26.3.1 / npm 12.0.1)
- **MongoDB Atlas** cluster (production only; not required in demo mode)

---

## Local setup

### 1. Clone the repository

```bash
git clone https://github.com/ThisSideMahesh/internship-guidance-agent.git
cd internship-guidance-agent
```

### 2. Configure the backend

Copy the example and fill in your values:

```bash
cp backend/.env.example backend/.env
```

Minimum required for demo mode (no MongoDB needed):

```env
DEMO_MODE=true
SEED_DEMO_DATA=true
JWT_SECRET=replace-with-a-random-string-of-at-least-32-characters
```

For a persistent MongoDB deployment, add:

```env
DEMO_MODE=false
MONGODB_URI=mongodb+srv://USERNAME:PASSWORD@YOUR_CLUSTER.mongodb.net/
MONGODB_DATABASE=internship_guidance
```

For AI features, add one of:

```env
OPENAI_API_KEY=sk-...
# or
GEMINI_API_KEY=AIza...
# or
GROQ_API_KEY=gsk_...
```

If no key is set, the assistant falls back to a deterministic mock response. Set `LLM_MODE=auto` (the default) to let the app pick whichever key is present.

> **Security:** Never commit `backend/.env`. It is listed in `.gitignore`. Keep all secrets in the local file only.

### 3. Create the Python virtual environment

```bash
python3.11 -m venv .venv --system-site-packages
.venv/bin/pip install -r backend/requirements.txt
```

### 4. Configure the frontend

```bash
cp frontend-react/.env.example frontend-react/.env.local
```

The default value (`VITE_API_BASE_URL=http://127.0.0.1:8000`) works for local development without changes.

### 5. Install frontend dependencies

```bash
cd frontend-react
npm install
```

---

## Running the application

Open two terminal windows from the project root.

**Terminal 1 — Backend**

```bash
.venv/bin/python -m uvicorn backend.main:app --host 127.0.0.1 --port 8000 --reload
```

**Terminal 2 — Frontend**

```bash
cd frontend-react
npm run dev
```

| Service | URL |
|---|---|
| React frontend | http://localhost:5173 |
| FastAPI health check | http://127.0.0.1:8000/api/health |
| Interactive API docs | http://127.0.0.1:8000/docs |

The health endpoint returns:

```json
{"status": "ok", "database": "demo", "demo_mode": true, "llm_mode": "mock"}
```

---

## Optional: seed internship data

Seed India-focused listings (Amazon, Microsoft, Adobe, Salesforce, and others):

```bash
.venv/bin/python -m backend.scripts.seed_india_internships
```

Sync recent listings from The Muse public feed:

```bash
.venv/bin/python -m backend.scripts.sync_live_internships --pages 5 --max-age-days 365
```

These scripts require `DEMO_MODE=false` and a valid `MONGODB_URI`.

---

## Running tests

```bash
.venv/bin/python -m pytest backend/tests -q
```

Expected: **80 passed, 1 skipped**. The skipped test requires a live MongoDB connection.

---

## API documentation

FastAPI generates interactive docs automatically at:

- **Swagger UI:** http://127.0.0.1:8000/docs
- **ReDoc:** http://127.0.0.1:8000/redoc

Key endpoint groups:

| Prefix | Description |
|---|---|
| `/api/auth` | Register, login, logout, current user |
| `/api/profile` | Read and update student profile |
| `/api/internships` | Authenticated internship catalog |
| `/api/public/internships` | Unauthenticated catalog (no token required) |
| `/api/recommendations` | Ranked matches for logged-in user |
| `/api/favorites` | Save and unsave opportunities |
| `/api/applications` | Application tracker CRUD |
| `/api/agent/chat` | AI assistant chat endpoint |
| `/api/student` | Interview history, learning progress, GitHub evaluator |
| `/api/health` | Health and mode check |

---

## Demo mode limitations

- All data is stored in memory and lost when the server restarts.
- The AI assistant returns deterministic mock responses unless an LLM provider key is configured.
- Resume file tools work in demo mode but processed files are not persisted.
- The data sync scripts (`seed_india_internships`, `sync_live_internships`) require a real MongoDB connection.
- Safety signals on application cards are heuristics, not guarantees. Always verify an opportunity on the official employer page before applying.

---

## Environment variables reference

| Variable | Required | Description |
|---|---|---|
| `DEMO_MODE` | No (default `false`) | `true` enables in-memory storage with no MongoDB required |
| `SEED_DEMO_DATA` | No (default `true`) | Seeds demo internships on startup when `DEMO_MODE=true` |
| `MONGODB_URI` | Yes (production) | MongoDB Atlas connection string |
| `MONGODB_DATABASE` | No (default `internship_guidance`) | Database name |
| `JWT_SECRET` | Yes | Random string, minimum 32 characters. Auto-generated in demo mode |
| `JWT_EXPIRY_MINUTES` | No (default `60`) | Token lifetime in minutes |
| `CORS_ORIGINS` | No | Comma-separated list of allowed frontend origins |
| `LLM_MODE` | No (default `auto`) | `auto`, `mock`, `openai`, `gemini`, or `groq` |
| `OPENAI_API_KEY` | No | Required if `LLM_MODE=openai` |
| `GEMINI_API_KEY` | No | Required if `LLM_MODE=gemini` |
| `GROQ_API_KEY` | No | Required if `LLM_MODE=groq` |
