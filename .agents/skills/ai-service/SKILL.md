---
name: ai-service
description: >-
  Use this skill when working on the Python FastAPI AI microservice (apps/ai-service/).
  Covers HuggingFace sentence-transformers for embeddings, pgvector similarity search,
  scikit-learn pricing model, semantic search, fraud detection, and batch embedding scripts.
---

# AI Service Skill

## Project Location
All AI/ML code lives in `apps/ai-service/`.

## Key Commands
```bash
# Start AI service dev server
cd apps/ai-service && uv run uvicorn app.main:app --reload --port 8001

# Run tests
cd apps/ai-service && uv run pytest tests/ -v

# Batch embed all active listings (run after seeding DB)
cd scripts && uv run python embed_listings.py

# Train pricing model
cd apps/ai-service && uv run python app/ml/train/train_pricing.py

# Train fraud detection model
cd apps/ai-service && uv run python app/ml/train/train_fraud.py
```

## Embedding Model
- Model: `sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2`
- Output dimensions: **384**
- Supports: English + Hindi queries natively
- Stored in: `listings.embedding` column (pgvector `vector(384)`)

## Adding a New Listing Embedding (Standard Flow)
1. New listing created in core API (`apps/api/`)
2. Core API calls `POST /ai/embed` on AI service (internal call)
3. AI service generates embedding via `embedding_service.generate_embedding()`
4. Embedding saved to `listings.embedding` in Supabase via pgvector

## Recommendation Query Pattern
```python
# Always use cosine distance operator <=> for similarity
# Lower distance = more similar
SELECT id, title, 1 - (embedding <=> $1::vector) AS similarity
FROM listings
WHERE status = 'ACTIVE' AND id != $2
ORDER BY embedding <=> $1::vector
LIMIT 6;
```

## Semantic Search Pattern
```python
# 1. Encode user query
query_embedding = model.encode(user_query, normalize_embeddings=True)
# 2. Run pgvector search with optional PostGIS geo filter
# 3. Re-rank results by (0.7 * similarity + 0.3 * recency_score)
```

## ML Model Files
- Pricing model: `app/ml/models/pricing_gbm.pkl` (scikit-learn GradientBoostingRegressor)
- Fraud model: `app/ml/models/fraud_isolation_forest.pkl` (IsolationForest)
- Load once at startup via `lifespan` context manager — never reload per-request

## AI Service Endpoints (Internal — not public)
```
POST /ai/embed              → Generate + store embedding for a listing_id
POST /ai/recommend          → Get similar listings by listing_id
POST /ai/semantic-search    → NLP query → ranked listing IDs
POST /ai/price-suggestion   → Suggest daily rent range for owner
POST /ai/fraud-check        → Score new listing for spam/fraud risk
```

## Environment Variables Required
```
DATABASE_URL=postgresql+asyncpg://...
HUGGINGFACE_MODEL=sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2
MODEL_DIR=app/ml/models
```
