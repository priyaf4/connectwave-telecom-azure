# ConnectWave Telecom Assistant (Azure)

An AI chat assistant for a telecom company: it answers billing and account questions (from a billing API),
policy/plan questions (RAG over PDF documents) and weather questions, using OpenAI with tool calling.

| Part | Tech | Azure service |
|---|---|---|
| `frontend/` | Static HTML/JS/CSS | Static Web Apps |
| `backend/` (`main.py`) | FastAPI, OpenAI, Chroma RAG (`rag_local_chroma`) | Container Apps (public) |
| `backend/billing/` | FastAPI mock billing API | Container Apps (internal only) |
| `infra/main.bicep` | Infrastructure as code | ACR, Key Vault, Blob Storage, Log Analytics, Application Insights, managed identities |

**Deploy to Azure:** see [deploy/README.md](deploy/README.md) (3 commands, or automatic via GitHub Actions).

## Run locally
```bash
python -m venv .venv
.venv\Scripts\pip install -r backend\requirements.txt          # Windows  (Linux/macOS: .venv/bin/pip ...)
copy backend\.env.example backend\.env                         # then fill in OPENAI_API_KEY
```
Set `RAG_BACKEND=rag_local_chroma` in `backend/.env` (same as production; no Pinecone key needed).

Three terminals, from the project root:
```bash
cd backend && ..\.venv\Scripts\python.exe billing\billing_api.py      # billing API  :4000
cd backend && ..\.venv\Scripts\python.exe main.py                     # backend      :3000
cd frontend && ..\.venv\Scripts\python.exe -m http.server 8080        # frontend     http://localhost:8080
```

## Tests
```bash
cd backend
pip install pytest ruff
pytest            # 48 offline tests: API, billing API, tools, rate limiter, RAG helpers
ruff check .
python -m tests.test_questions    # manual end-to-end questions against the real LLM (needs keys + billing API)
```

## API
| Method | Path | Purpose |
|---|---|---|
| POST | `/ask` | `{"question": "...", "provider": "openai"}` → `{"answer", "tools_used", "provider", "request_id"}` |
| GET | `/health` | Liveness (process up, returns the deployed version) |
| GET | `/ready` | Readiness (configuration complete) |

Errors: `422` invalid question, `429` rate limited, `502` upstream failure, `504` timeout — always with a friendly
message and a `request_id` to find the details in the logs.

## Project notes
- Files ending in `_old` (e.g. `main_old.py`, `Dockerfile_old`, `deploy/README_old.md`) are the previous versions
  kept for reference. They are excluded from images, the website and linting.
- `Azure steps.txt` and `FileDetails.txt` describe the original POC; `deploy/README.md` is the current guide.
