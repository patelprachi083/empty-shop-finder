# AI Service Sub-directory Rules

## Python FastAPI — AI Microservice

### ML Model Rules
- All ML models loaded **once at startup** via `lifespan` context — never reload per-request.
- Model files (`.pkl`) stored in `app/ml/models/` — committed to git via Git LFS.
- When retraining a model, save to a new versioned filename (`pricing_gbm_v2.pkl`) and update config.

### Embedding Rules
- Always use `normalize_embeddings=True` when calling `model.encode()`.
- Embedding dimension is **384** (MiniLM-L12). Never change without a DB migration for the vector column.
- Batch embed listings using `scripts/embed_listings.py` — never embed one-by-one in a loop without batching.

### Performance Rules
- This service runs on **Cloud Run with scale-to-zero** — startup time must be < 30 seconds.
- Cache the HuggingFace model in the Docker image (download at build time, not runtime).
- All pgvector queries must use the `ivfflat` index — verify with `EXPLAIN ANALYZE`.

### Security Rules
- AI service endpoints are **internal only** — not exposed to the public internet.
- Access is restricted to the core API service via VPC connector or Cloud Run service-to-service auth.
