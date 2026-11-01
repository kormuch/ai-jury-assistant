# AI Jury Assistant

AI-powered hackathon pitch evaluator. Analyzes team repositories, listens to presentations via speech-to-text, generates follow-up tasks, and produces weighted scores across six categories — serving as one voice among human jurors.

## Quick Start

```bash
# Clone & install
python -m venv .venv
.venv/Scripts/activate          # Windows (.venv/bin/activate on Linux/Mac)
pip install -r requirements.txt

# Configure
cp .env.example .env
# Add your GROQ_API_KEY or GEMINI_API_KEY

# Run
python -m backend.main
```

Open **http://127.0.0.1:8000** in your browser.

## How It Works

```
  Team submits repo
        |
        v
  +--------------+     +-----------+     +---------------+
  | Repo Analyzer|---->| LLM Eval  |---->| Task Gen      |
  | (clone+scan) |     | (scores)  |     | (2-3 tasks)   |
  +--------------+     +-----------+     +-------+-------+
                                                 |
  Team presents (5 min)                          |
        |                                        |
        v                                        v
  +-----------+     +---------------+     +-----------+
  | Mic->STT  |---->| Transcript    |     | TTS       |
  | (Whisper) |     | (live feed)   |     | speaks    |
  +-----------+     +---------------+     | task      |
                                          +-----------+
                                                 |
  Team works on task (timeboxed)                 |
        |                                        |
        v                                        v
  +--------------------------------------------------+
  | Final Evaluation                                  |
  | repo scores + presentation + task completion      |
  | -> human review -> submit                         |
  +--------------------------------------------------+
```

## Scoring Rubric

Each category scored 1-10 with weighted contribution to the final score:

| Category | Weight | Evaluates |
|----------|--------|-----------|
| Architecture & Design | 25% | Structure, separation of concerns, tech choices |
| Code Quality | 20% | Readability, consistency, error handling |
| Completeness | 20% | Working features vs stubs/placeholders |
| Innovation | 15% | Creative approaches, smart tool usage |
| Documentation | 10% | README quality, API docs, setup instructions |
| Presentation | 10% | Clarity, structure, demo quality (from transcript) |

## Project Structure

```
backend/
  config.py           # Settings from .env (API keys, LLM provider, host/port)
  database.py         # SQLite via aiosqlite (teams, analyses, transcripts, evaluations)
  repo_analyzer.py    # Clone, scan structure, read key files, build analysis dict
  llm_service.py      # Groq/Gemini dual provider, scoring prompts, task generation
  tts_service.py      # Edge TTS (German voice)
  main.py             # FastAPI app, all API endpoints
frontend/
  index.html          # SPA shell
  style.css           # Dark theme, score bars, timer, badges
  app.js              # Full SPA logic (~900 lines)
```

## API Reference

Base URL: `http://127.0.0.1:8000/api`

### Teams

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/teams` | Create team `{name, repo_url}` |
| `GET` | `/teams` | List all teams |
| `GET` | `/teams/{id}` | Get team with analysis + evaluation data |
| `DELETE` | `/teams/{id}` | Delete team and all related data |

### Repo Analysis

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/teams/{id}/analyze` | Start repo analysis (background task) |
| `GET` | `/teams/{id}/analysis` | Get analysis results (scores, tasks) |

### Transcript

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/teams/{id}/transcript` | Append transcript text `{text}` |
| `GET` | `/teams/{id}/transcript` | Get full transcript |
| `WS` | `/ws/stt/{team_id}` | WebSocket for live STT |

### Tasks

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/teams/{id}/tasks` | Get generated follow-up tasks |
| `POST` | `/teams/{id}/assign-task` | Assign task `{task_index}` |
| `POST` | `/teams/{id}/task-tts` | Generate TTS audio for assigned task |

### Evaluation

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/teams/{id}/evaluate` | Generate final evaluation (background task) |
| `GET` | `/teams/{id}/evaluation` | Get evaluation results |
| `PUT` | `/teams/{id}/evaluation` | Update scores/justifications (human edits) |
| `POST` | `/teams/{id}/submit` | Mark evaluation as submitted |

### Results

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/results` | All teams ranked by final score |
| `GET` | `/results/export` | Markdown export of all evaluations |

### TTS

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/tts` | Generate TTS for any text `{text}` |

## Configuration

All settings via `.env`:

```env
GROQ_API_KEY=gsk_...              # Required if LLM_PROVIDER=groq
GEMINI_API_KEY=AI...              # Required if LLM_PROVIDER=gemini
LLM_PROVIDER=groq                 # "groq" or "gemini"
LLM_MODEL=llama-3.3-70b-versatile # Model identifier
HOST=127.0.0.1
PORT=8000
REPOS_DIR=./repos
```

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Backend | Python 3.12, FastAPI, uvicorn |
| Frontend | Vanilla HTML/CSS/JS |
| Database | SQLite via aiosqlite |
| LLM | Groq (Llama 3.3 70B) or Google Gemini |
| TTS | Microsoft Edge TTS |
| STT | Whisper API / Deepgram |

## Team Status Flow

```
pending -> analyzing -> analyzed -> presenting -> task_assigned -> evaluated -> submitted
               |
             error
```
