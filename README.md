# AI Image Understanding & Content Matching Engine

An AI-powered backend that understands image content, matches images to written posts using semantic similarity, and rejects incorrect recommendations through a mismatch guard.

## Problem

Content teams may have many images and written posts but still need to manually decide which image belongs with which post. A simple similarity search can return a visually or semantically related image that is still factually wrong.

Example:

- Post: `The behavior of red foxes`
- Candidate A: red fox image
- Candidate B: gray wolf image
- A similarity-only system may rank the wolf highly.
- This system adds a mismatch guard that checks the candidate against the expected subject and confidence before recommending it.

## Core Features

1. **Image understanding** — process an image with a vision model and store validated metadata such as subject, category, attributes, caption, and confidence.
2. **Semantic matching** — embed image descriptions and post content, then rank candidate images by semantic similarity.
3. **Mismatch guard** — reject candidates that fail subject/category/confidence checks and explain the rejection.
4. **Async AI processing** — run vision and embedding work in background jobs instead of blocking API requests.
5. **Human review** — approve or reject recommendations and inspect why a candidate was selected or rejected.

## Architecture

```text
Image Upload ──► API ──► Database
                      │
                      └──► Background Worker
                              │
                              ├── Vision Model
                              │       └── Structured Metadata
                              │
                              └── Embedding Model
                                      └── Vector Storage

Post ─────────► Embedding ───────────────┐
                                         ▼
                                  Similarity Ranking
                                         │
                                         ▼
                                  Mismatch Guard
                                    /          \
                                  PASS         FAIL
                                   │             │
                                   ▼             ▼
                              Suggestion     Explanation
                                   │
                                   ▼
                              Human Review
```

## Tech Stack

- Python + FastAPI
- PostgreSQL
- SQLAlchemy + Alembic
- Redis + Celery
- Vision model for structured image understanding
- Embedding model + pgvector
- Pydantic for response validation
- Pytest

## Planned API

```text
POST /api/v1/images
GET  /api/v1/images/{id}
POST /api/v1/posts
GET  /api/v1/posts/{id}/images
GET  /api/v1/suggestions/{id}
POST /api/v1/suggestions/{id}/approve
POST /api/v1/suggestions/{id}/reject
GET  /health
```

## Project Structure

```text
app/
├── api/v1/       # HTTP routes
├── core/         # configuration and shared application logic
├── db/           # database session and setup
├── models/       # SQLAlchemy models
├── schemas/      # Pydantic request/response schemas
├── services/     # matching, AI, guard, and review logic
├── workers/      # Celery/background tasks
└── main.py       # FastAPI application

tests/            # automated tests
seed_data/        # reproducible demo images/posts/data
scripts/          # setup and seed helpers
docs/             # design notes and evidence
```

## Development Status

**Phase:** Initial setup

The repository currently contains the project skeleton and health endpoint. AI processing, matching, the mismatch guard, and review workflow will be implemented incrementally.

## Local Setup

```bash
git clone <repository-url>
cd ai-image-matching-engine
python -m venv .venv
# Windows
.venv\\Scripts\\activate
# macOS/Linux
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
```

Run the API:

```bash
uvicorn app.main:app --reload
```

Health check:

```text
GET http://localhost:8000/health
```

## Scope

The core system is intentionally small: one vision model, one embedding model, a small image corpus, a matching pipeline, a mismatch guard, and a review API. No full frontend is required.

## Capstone Concepts

| Concept | Implementation |
|---|---|
| API endpoints | FastAPI routes for images, posts, suggestions, and review |
| Database | PostgreSQL persistence for image metadata, posts, embeddings, and suggestions |
| Background jobs | Celery + Redis for asynchronous AI processing |
| LLM integration | Vision model produces validated structured image metadata |
| Authentication | Planned for protected management/review endpoints |

## 10x Goal

Reduce the time needed to find a suitable image for a written post by automatically understanding the content, ranking candidates, and rejecting incorrect matches.

## Important Rules

- Keep API keys and credentials in environment variables.
- Do not commit `.env` or secrets.
- Use a small reproducible demo dataset.
- Keep the core implementation within the capstone scope.
