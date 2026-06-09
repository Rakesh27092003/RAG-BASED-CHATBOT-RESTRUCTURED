## RAG Chatbot Backend

This repository contains a Streamlit frontend stub and a FastAPI backend scaffold for a retrieval-augmented generation (RAG) chatbot.

### Structure

- `backend/`
	- `configs/` — pydantic settings loaded from `.env`
	- `models/` — request and response schemas
	- `services/` — ingestion, embeddings, indexing, search, and generation logic
	- `routers/` — FastAPI route definitions
	- `utils/` — model routing, chunking, and helper utilities
- `frontend/` — lightweight Streamlit frontend stub
- `uploaded_files/` — persisted uploaded documents and index data

### Setup

1. Create a virtual environment and install dependencies:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

2. Copy the environment example and set provider keys if needed:

```powershell
copy .env.example .env
```

3. Start the backend server:

```powershell
uvicorn backend.main:app --reload --host 0.0.0.0 --port 8000
```

4. Optionally run the frontend stub:

```powershell
# RAG Chatbot (Streamlit frontend + REST backend)

This repository provides a lightweight Streamlit frontend that delegates document processing, indexing, embeddings, and generation to a separate FastAPI backend.

This README reflects the current code: the frontend is a thin client and the backend implements the heavy lifting.

---

## Contents

- `frontend/streamlit_app.py` — Streamlit UI that uploads documents and sends chat queries to the backend.
- `backend/` — FastAPI service providing endpoints to process documents, search, and generate answers.
- `uploaded_files/` — runtime storage for uploaded documents and index artifacts.

## Quick start

1. Install dependencies (virtualenv recommended):

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

2. Start the backend:

```powershell
uvicorn backend.main:app --reload --host 0.0.0.0 --port 8000
```

3. Start the frontend:

```powershell
streamlit run frontend/streamlit_app.py
```

4. In the Streamlit sidebar set `Backend URL` if your backend is not at `http://localhost:8000`.

---

## Backend API (key endpoints)

- `POST /documents/process`
	- Form fields: `file` (multipart upload) or `filename` (existing file in storage), optional `chunk_size`, `chunk_overlap`, `index_name`.
	- Returns: `{ "index_name": <str>, "num_chunks": <int>, "index_path": <str> }`.

- `POST /generate`
	- Body JSON: `{ "prompt": <str>, "top_k": <int> }`.
	- Returns: `{ "answer": <str>, "source_chunks": [ {"chunk_id":..., "score":..., "text":"..."}, ... ] }`.

- `POST /search`
	- Body JSON: `{ "query": <str>, "top_k": <int> }`.
	- Returns retrieval results from the single shared index.

- `GET /indexes/current`
	- Returns metadata about the shared index: `{ "index_exists": bool, "index_name": str, "num_chunks": int, "use_faiss": bool }`.

---

## Configuration

Set environment variables or a `.env` file to configure the backend (see `backend/configs/model_config.py`):

- `FILE_STORAGE_PATH` (default `./uploaded_files`) — where uploaded files and indexes are stored.
- `SINGLE_INDEX_NAME` (default `main_index`) — shared index name used by the backend.
- Provider and API keys: `EMBEDDING_PROVIDER`, `GENERATION_PROVIDER`, `GEMINI_API_KEY`, `ANTHROPIC_API_KEY`, etc.

---

## Notes & implementation details

- The backend uses provider-specific model routing (`backend.utils.model_routing`) to call external LLM/embedding services.
- Index persistence uses FAISS when available; otherwise compressed NumPy storage is used.
- The app is built around a single shared index; the frontend does not request or manage multiple index names.

---

## Troubleshooting

- If `/generate` returns a 422 error, ensure the request JSON contains `prompt` (backend expects `prompt`).
- If `indexes/current` reports `index_exists: false`, upload and process a document via the frontend or `POST /documents/process`.

---
